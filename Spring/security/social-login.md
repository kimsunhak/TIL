## Spring Social-Login 정리



### CORS 활성화

> FrontEnd Client가 어플리케이션에 접근 할 수 있도록 CORS를 활성화
>
> 아래의 설정은 모든 Origin을 활성화하므로 상용에서는 수정해서 사용

##### WebMvcConfig.class

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    private final long MAX_AGE_SECONDS = 3600;

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "POST", "PATCH", "DELETE", "PUT", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(MAX_AGE_SECONDS);
    }
}
```



### SecurityConfig 설정

> SecurityConfig 는 보안 구현의 핵심, OAuth2 Social 로그인과 이메일 및 비밀번호 기반 로그인에 대한 구성을 적용
>
> - csrf().disable()
>   - Jwt token을 사용할 것이므로 비활성화 한다.
>   - 일반 사용자들이 브라우저를 통해 요청을 하는 경우에는 CSRF 방지를 사용하는 것이 좋다.
> - httpBasic().disable()
>   - 요청 헤더에 username와 password를 실어 보내면 브라우저 또는 서버가 그 값을 읽어서 인증하는 방식인 Basic 인증을 비활성화 한다.

```java
@Configuration
@RequiredArgsConstructor
@EnableWebSecurity
@EnableGlobalMethodSecurity(
        securedEnabled = true,
        jsr250Enabled = true,
        prePostEnabled = true
)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomUserDetailsService customUserDetailsService;

    private final CustomOAuth2UserService customOAuth2UserService;

    private final OAuth2AuthenticationSuccessHandler oAuth2AuthenticationSuccessHandler;

    private final OAuth2AuthenticationFailureHandler oAuth2AuthenticationFailureHandler;

    private final HttpCookieOAuth2AuthorizationRequestRepository httpCookieOAuth2AuthorizationRequestRepository;

    private final TokenAuthenticationFilter tokenAuthenticationFilter;


    private static final String[] AUTH_WHITELIST = {
            // Swagger UI
            "/v2/api-docs",
            "/swagger-resources",
            "/swagger-resources/**",
            "/configuration/ui",
            "/configuration/security",
            "/swagger-ui.html",
            // static resources
            "/webjars/**",
            "/",
            "/error",
            "/favicon.ico",
            "/**/**.css",
            "/**/**.js",
            "/**/**.png",
            "/**/**.jpg",
            "/**/**.html",
            "/**/**.woff",
            "/**/**.otf",
            "/**/**.eot",
            // uri
            "/auth/**",
            "/redirect"
    };

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public HttpCookieOAuth2AuthorizationRequestRepository cookieOAuth2AuthorizationRequestRepository() {
        return new HttpCookieOAuth2AuthorizationRequestRepository();
    }

    @Bean(BeanIds.AUTHENTICATION_MANAGER)
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder authenticationManagerBuilder) throws Exception {
        authenticationManagerBuilder
                .userDetailsService(customUserDetailsService)
                .passwordEncoder(passwordEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .cors()
                .and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .csrf()
                    .disable()
                .httpBasic()
                    .disable()
                .formLogin()
                    .disable()
                .exceptionHandling()
                    .authenticationEntryPoint(new RestAuthenticationEntryPoint())
                    .and()
                .authorizeRequests()
                    .antMatchers(AUTH_WHITELIST)
                    .permitAll()
                .antMatchers(
                        "/api/**")
                .permitAll()
                .anyRequest()
                    .authenticated()
                    .and()
                .oauth2Login()
                    .authorizationEndpoint()
                        .baseUri("/oauth2/authorize")
                        .authorizationRequestRepository(httpCookieOAuth2AuthorizationRequestRepository)
                        .and()
                    .redirectionEndpoint()
                        .baseUri("/oauth2/callback/*")
                        .and()
                    .userInfoEndpoint()
                        .userService(customOAuth2UserService)
                        .and()
                    .successHandler(oAuth2AuthenticationSuccessHandler)
                    .failureHandler(oAuth2AuthenticationFailureHandler);
        http.addFilterBefore(tokenAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
    }
}
```



#### AppProperties 바인딩 방법

> @ConfigurationProperties 란?
>
> *.properties, *.yml 파일에 있는 property를 자바 클래스에 값을 가져와서 사용할 수 있게 해주는 어노테이션이다.
>
> @ConfigurationProperties 를 사용해 app 으로 시작하는 property를 POJO class로 바인딩

##### config / AppProperties.class

~~~java
@Getter
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private final Auth auth = new Auth();
    private final OAuth2 oAuth2 = new OAuth2();

    @Getter
    @Setter
    public static class Auth {
        private String tokenSecret;
        private long refreshTokenExpirationMsec;
        private long accessTokenExpirationMsec;
    }

    public static final class OAuth2 {
        private List<String> authorizedRedirectUris = new ArrayList<>();

        public List<String> getAuthorizedRedirectUris() {
            return authorizedRedirectUris;
        }

        public OAuth2 authorizedRedirectUris(List<String> authorizedRedirectUris) {
            this.authorizedRedirectUris = authorizedRedirectUris;
            return this;
        }
    }
}
~~~

##### *.properties

```
# 예시
app.naver=https://www.naver.com
```



#### AppProperties 활성화

> @EnableConfigurationProperties 을 사용하여 AppProperties를 활성화해야 사용가능

##### MainApplication.class

~~~java
@EnableConfigurationProperties(AppProperties.class)
public class MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}
~~~

------

### JWT 설정

`package : security `



#### JWT TokenProvider

> JSON 웹 토큰을 생성하고 확인하는 코드를 만듬
>
> .claim : Token에 담을 정보
> .setSubject : Token 제목
> .setIssuedAt : 토큰 발급 시간
> .setExpiration : 토큰 만료 시간
> .signWith : 사용할 알고리즘과, 시크릿값 셋팅
> .compact : 토큰 생성

##### security / TokenProvider.class

```java
@Service
@RequiredArgsConstructor
public class TokenProvider {

    private static final Logger logger = LoggerFactory.getLogger(TokenProvider.class);

    private final AppProperties appProperties;

    private final MemberRepository memberRepository;

    /**
     * UserToken 토큰 생성, 만료시간 X
     * @param authentication
     * @return
     */
    public Token createToken(Authentication authentication) {
        UserPrincipal userPrincipal = (UserPrincipal) authentication.getPrincipal();

        Date now = new Date();
        Date accessExpiryDate = new Date(now.getTime() + appProperties.getAuth().getAccessTokenExpirationMsec());
        String jwtSecret = Base64Utils.encodeToString(appProperties.getAuth().getTokenSecret().getBytes());

        String accessToken = Jwts.builder()
                .setSubject(Long.toString(userPrincipal.getId()))
                .claim("type", "access")
                .claim("role", authentication.getAuthorities().stream().map(GrantedAuthority::getAuthority).collect(Collectors.joining(",")))
                .setIssuedAt(new Date())
                .setExpiration(accessExpiryDate)
                .signWith(SignatureAlgorithm.HS512, jwtSecret)
                .compact();

        return new Token(accessToken, accessExpiryDate);
    }

    /**
     * UserToken 토큰 생성, 만료시간 O
     * @param authentication
     * @param expiryDate
     * @return
     */
    public Token createToken(Authentication authentication, long expiryDate) {
        UserPrincipal userPrincipal = (UserPrincipal) authentication.getPrincipal();

        Date now = new Date();
        Date accessExpiryDate = new Date(now.getTime() + appProperties.getAuth().getAccessTokenExpirationMsec());
        String jwtSecret = Base64Utils.encodeToString(appProperties.getAuth().getTokenSecret().getBytes());

        String accessToken = Jwts.builder()
                .setSubject(Long.toString(userPrincipal.getId()))
                .claim("type", "access")
                .claim("role", authentication.getAuthorities().stream().map(GrantedAuthority::getAuthority).collect(Collectors.joining(",")))
                .setIssuedAt(now)
                .setExpiration(accessExpiryDate)
                .signWith(SignatureAlgorithm.HS512, jwtSecret)
                .compact();

        return new Token(accessToken, accessExpiryDate);
    }

    /**
     * Token 유효성검사
     * @param token
     */
    public void validateToken(String token) {
        String jwtSecret = Base64Utils.encodeToString(appProperties.getAuth().getTokenSecret().getBytes());
        Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token);
    }

    /**
     * AccessToken 유효성검사
     * @param accessToken
     */
    public void validateAccessToken(String accessToken) {

        if (!getTypeFromToken(accessToken).equals("access")) {
            throw new JwtException("토큰 타입이 불일치 합니다. access Token이 필요합니다", "404");
        }

        try {
            validateToken(accessToken);
        } catch (SignatureException exception) {
            throw new JwtException("Invalid JWT signature", "400");
        } catch (MalformedJwtException exception) {
            throw new JwtException("Invalid JWT token", "401");
        } catch (ExpiredJwtException exception) {
            throw new JwtException("Expired JWT token", "402");
        } catch (UnsupportedJwtException exception) {
            throw new JwtException("Unsupported JWT token", "403");
        }
    }

    /**
     * Token에 UserId값 조회
     * @param token
     * @return
     */
    public Long getMemberIdFromToken(String token) {
        String jwtSecret = Base64Utils.encodeToString(appProperties.getAuth().getTokenSecret().getBytes());
        Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();

        return Long.parseLong(claims.getSubject());
    }

    /**
     * Token에 Type값 조회
     * @param token
     * @return
     */
    public String getTypeFromToken(String token) {
        String jwtSecret = Base64Utils.encodeToString(appProperties.getAuth().getTokenSecret().getBytes());
        Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();
        return claims.get("type").toString();
    }

    /**
     * Token에 Role값 조회
     * @param token
     * @return
     */
    public String getRoleFromToken(String token) {
        String jwtSecret = Base64Utils.encodeToString(appProperties.getAuth().getTokenSecret().getBytes());
        Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();

        return claims.get("role").toString();
    }

    /**
     * Header에 토큰값이 있는지 확인
     * @param request
     * @return
     */
    public String getJwtFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");

        if (!StringUtils.hasText(bearerToken)) {
            return null;
        }

        if (!bearerToken.startsWith("Bearer ")) {
            throw new RequestParamException("jwt Token은 Bearer로 시작해야 합니다", "402");
        }

        return bearerToken.substring(7);
    }

}


```



#### JWT TokenAuthenticationFilter

> JWT 인증 토큰을 읽고, 확인하고, SecurityContext 토큰이 유효한 경우 Spring Securirty를 설정 하는데 사용
>
> UserPasswordAuthenticationToken : 사용자 인증용 객체
>
> authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request)) : 서버 측의 현재 요청에 대한 세션 정보를로드

```java
@Component
@RequiredArgsConstructor
public class TokenAuthenticationFilter extends OncePerRequestFilter {

    private final TokenProvider tokenProvider;

    private final CustomUserDetailsService customUserDetailsService;

    private final ObjectMapper objectMapper;

    private static final Logger logger = LoggerFactory.getLogger(TokenAuthenticationFilter.class);

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        try {
            String jwt = tokenProvider.getJwtFromRequest(request);

            if (StringUtils.hasText(jwt)) {
//                Long userId = tokenProvider.getMemberIdFromToken(jwt);
                tokenProvider.validateAccessToken(jwt);
                authMember(request, jwt);
            }
        } catch (JwtException e) {
            logger.error(e.getMessage());
            jwtResponse(response, e.getErrorCode(), e.getMessage());
            return;
        } catch (ResourceNotFoundException e) {
            logger.error(e.getMessage());
            jwtResponse(response, e.getErrorCode(), e.getMessage());
            return;
        }

        filterChain.doFilter(request, response);
    }

    /**
     * Jwt 토큰으로 인증정보 조회
     * @param request
     * @param jwt
     */
    private void authMember(HttpServletRequest request, String jwt) {
        UserDetails userDetails = loadUserByJwt(jwt);
        UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
        authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

        SecurityContextHolder.getContext().setAuthentication(authentication);
    }

    private void jwtResponse(HttpServletResponse response, String errorCode, String message) throws IOException {
        ApiResponse apiResponse = new ApiResponse(false, errorCode, message);
        String responseString = objectMapper.writeValueAsString(apiResponse);
        response.setContentType("application/json;charset=UTF-8");
        logger.info("responseString : " + responseString);
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        response.getWriter().write(responseString);
    }

    /**
     * JWT 토큰으로 DB 정보 조회
     * @param jwt
     * @return
     */
    private UserDetails loadUserByJwt(String jwt) {
        String role = tokenProvider.getRoleFromToken(jwt);
        if (role.equals("ROLE_MEMBER")) {
            return customUserDetailsService.loadUserById(tokenProvider.getMemberIdFromToken(jwt));
        } else if (role.equals("ROLE_ADMIN")) {
            return customUserDetailsService.loadUserById(tokenProvider.getMemberIdFromToken(jwt));
        }
        return null;
    }
}
```



#### JWT RestAuthenticationEntryPoint

> 사용자가 인증없이 보호 된 리소스에 액세스하려고 할 때 호출됩니다. 이 경우 401 Unauthorized 응답을 반환

```java
public class RestAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, authException.getLocalizedMessage());
    }
}
```



------

## OAuth2 설정



### OAuth2 인증을위한 사용자 정의 클래스

#### OAuth2UserInfo 매핑

> 인증 된 사용자의 세부 정보를 가져올 때 모든 OAuth2 공급자는 다른 JSON 응답을 반환
>
> 스프링 보안은 일반적인 `map`키-값 쌍 의 형태로 응답을 구문 분석

##### OAuth2UserInfo.class

> 일반 `map`키-값 쌍 에서 사용자의 필수 세부 정보를 가져 오는 데 사용

```java
public abstract class OAuth2UserInfo {
    protected Map<String, Object> attributes;

    public OAuth2UserInfo(Map<String, Object> attributes) {
        this.attributes = attributes;
    }

    public Map<String, Object> getAttributes() {
        return attributes;
    }

    public abstract String getId();

    public abstract String getName();

    public abstract String getEmail();
}
```

##### GoogleOAuth2UserInfo.class

```java
public class GoogleOAuth2UserInfo extends OAuth2UserInfo{

    public GoogleOAuth2UserInfo(Map<String, Object> attributes) {
        super(attributes);
    }

    @Override
    public String getId() {
        return (String) attributes.get("sub");
    }

    @Override
    public String getName() {
        return (String) attributes.get("name");
    }

    @Override
    public String getEmail() {
        return (String) attributes.get("email");
    }

    @Override
    public String getImageUrl() {
        return (String) attributes.get("picture");
    }
}
```



#### HttpCookieOAuth2AuthorizationRequestRepository

> OAuth2 프로토콜은 `state`CSRF 공격을 방지하기 위해 매개 변수 사용을 권장
>
> 인증 중에 애플리케이션은 인증 요청에서이 매개 변수를 전송하고 OAuth2 공급자는 OAuth2 콜백에서 변경되지 않은이 매개 변수를 반환
>
> 응용 프로그램은 `state`OAuth2 공급자에서 반환 된 매개 변수의 값을 초기에 보낸 값과 비교하고 일치하지 않으면 인증 요청을 거부
>
> 이 흐름을 얻으려면 애플리케이션이 `state`매개 변수를 어딘가에 저장하여 나중에 `state`OAuth2 공급자에서 반환 된 것과 비교할 수 있도록 해야 하고 쿠키에 `state` 뿐만 아니라 `redirect_uri` 도 저장
>
> HttpCookieOAuth2AuthorizationRequestRepository 인증 요청을 쿠키에 저장하고 검색하는 기능을 제공

##### HttpCookieOAuth2AuthorizationRequestRepository.class

```java
@Component
public class HttpCookieOAuth2AuthorizationRequestRepository implements AuthorizationRequestRepository<OAuth2AuthorizationRequest> {
    public static final String OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME = "oauth2_auth_request";
    public static final String REDIRECT_URI_PARAM_COOKIE_NAME = "redirect_uri";
    private static final int cookieExpireSeconds = 180;

    @Override
    public OAuth2AuthorizationRequest loadAuthorizationRequest(HttpServletRequest request) {
        return CookieUtils.getCookie(request, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME)
                .map(cookie -> CookieUtils.deserialize(cookie, OAuth2AuthorizationRequest.class))
                .orElse(null);
    }

    @Override
    public void saveAuthorizationRequest(OAuth2AuthorizationRequest authorizationRequest, HttpServletRequest request, HttpServletResponse response) {
        if (authorizationRequest == null) {
            CookieUtils.deleteCookie(request, response, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME);
            CookieUtils.deleteCookie(request, response, REDIRECT_URI_PARAM_COOKIE_NAME);
        }

        CookieUtils.addCookie(response, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME, CookieUtils.serialize(authorizationRequest), cookieExpireSeconds);
        String redirectUriAfterLogin = request.getParameter(REDIRECT_URI_PARAM_COOKIE_NAME);
        if (StringUtils.isNotBlank(redirectUriAfterLogin)) {
            CookieUtils.addCookie(response, REDIRECT_URI_PARAM_COOKIE_NAME, redirectUriAfterLogin, cookieExpireSeconds);
        }
    }

    @Override
    public OAuth2AuthorizationRequest removeAuthorizationRequest(HttpServletRequest request) {
        return this.loadAuthorizationRequest(request);
    }

    public void removeAuthorizationRequestCookies(HttpServletRequest request, HttpServletResponse response) {
        CookieUtils.deleteCookie(request, response, OAUTH2_AUTHORIZATION_REQUEST_COOKIE_NAME);
        CookieUtils.deleteCookie(request, response, REDIRECT_URI_PARAM_COOKIE_NAME);
    }
}
```



#### CustomOAuth2UserService

> `CustomOAuth2UserService` 는 Spring Security의 `DefaultOAuth2UserService` 상속하고 `loadUser()` 메소드를 구현
> 이 메소드는 OAuth2 공급자로부터 액세스 토큰을 얻은 후에 호출
>
> 먼저 OAuth2 공급자로부터 사용자의 세부 정보를 가져오고 동일한 이메일을 사용하는 사용자가 이미 데이터베이스에 있는 경우 세부 정보를 업데이트하고 그렇지 않으면 새 사용자를 등록

##### CustomOAuth2UserService.class

```java
@Service
@RequiredArgsConstructor
public class CustomOAuth2UserService extends DefaultOAuth2UserService {

    private final MemberRepository memberRepository;
    private final AppProperties appProperties;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest oAuth2UserRequest) throws OAuth2AuthenticationException {
        OAuth2User oAuth2User = super.loadUser(oAuth2UserRequest);

        try {
            return processOAuth2User(oAuth2UserRequest, oAuth2User);
        } catch (AuthenticationException e) {
            throw e;
        } catch (Exception e) {
            throw new InternalAuthenticationServiceException(e.getMessage(), e.getCause());
        }
    }

    private OAuth2User processOAuth2User(OAuth2UserRequest oAuth2UserRequest, OAuth2User oAuth2User) {
        OAuth2UserInfo oAuth2UserInfo = OAuth2UserInfoFactory.getOAuth2UserInfo(oAuth2UserRequest.getClientRegistration().getRegistrationId(), oAuth2User.getAttributes());

        if (StringUtils.isEmpty(oAuth2UserInfo.getEmail())) {
            throw new OAuth2AuthenticationProcessingException("OAuth2 Provider에 이메일이 없습니다.");
        }

        Optional<Member> memberOptional = memberRepository.findByEmail(oAuth2UserInfo.getEmail());

        Member member;
        if (memberOptional.isPresent()) {
            member = memberOptional.get();
            if (!member.getProvider().equals(AuthProvider.valueOf(oAuth2UserRequest.getClientRegistration().getRegistrationId()))) {
                throw new OAuth2AuthenticationProcessingException("이미 등록된 회원입니다.");
            }

            member = updateExistingMember(member, oAuth2UserInfo);
        } else {
            member = registerNewMember(oAuth2UserRequest, oAuth2UserInfo);
        }

        return UserPrincipal.create(member, oAuth2User.getAttributes());
    }

    private Member registerNewMember(OAuth2UserRequest oAuth2UserRequest, OAuth2UserInfo oAuth2UserInfo) {
        AuthProvider authProvider = AuthProvider.valueOf(oAuth2UserRequest.getClientRegistration().getRegistrationId());

        Member member = Member.builder()
                .name(oAuth2UserInfo.getName())
                .email(oAuth2UserInfo.getEmail())
                .provider(authProvider)
                .providerId(oAuth2UserInfo.getId())
                .imageUrl(oAuth2UserInfo.getImageUrl())
                .build();

        return memberRepository.save(member);
    }

    private Member updateExistingMember(Member member, OAuth2UserInfo oAuth2UserInfo) {
        member.updateExistingMember(oAuth2UserInfo.getName(), oAuth2UserInfo.getImageUrl());
        return memberRepository.save(member);
    }

}
```



#### OAuth2AuthenticationSuccessHandler

> 인증에 성공하면 Spring Security는 `SecurityConfig` 에 구성된 `OAuth2AuthenticationSuccessHandler` 의 `onAuthenticationSuccess()` 메소드를 호출
>
> 이 방법에서는 몇 가지 유효성 검사를 수행하고 JWT 인증 토큰을 만들고 재발급을 위한 refreshToken을 만들어 쿼리 문자열에 추가 된 JWT 토큰을 사용하여 클라이언트가 지정한 redirect_uri로 사용자를 리디렉션함

##### OAuth2AuthenticationSuccessHandler.class

```java
@Component
@RequiredArgsConstructor
public class OAuth2AuthenticationSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {

    private final AuthService authService;
    private final HttpCookieOAuth2AuthorizationRequestRepository httpCookieOAuth2AuthorizationRequestRepository;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) throws IOException, ServletException {
        String targetUrl = determineTargetUrl(request, response, authentication);

        if (response.isCommitted()) {
            logger.debug("Response has already been committed. Unable to redirect to " + targetUrl);
        }

        clearAuthenticationAttributes(request, response);
        getRedirectStrategy().sendRedirect(request, response, targetUrl);
    }

    protected String determineTargetUrl(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        SimpleDateFormat formatter = new SimpleDateFormat ( "yyyy/MM/dd HH:mm:ss", Locale.KOREA );
        Optional<String> redirectUri = CookieUtils.getCookie(request, REDIRECT_URI_PARAM_COOKIE_NAME).map(Cookie::getValue);

        if (!redirectUri.isPresent()) {
            throw new BadRequestException("인증되지 않은 REDIRECT_URI입니다.");
        }

        String targetUri = redirectUri.orElse(getDefaultTargetUrl());

        UserPrincipal userPrincipal = (UserPrincipal) authentication.getPrincipal();
        AuthResponse accessAndRefreshToken = authService.createAccessAndRefreshToken(userPrincipal.getId());
        Token accessToken = accessAndRefreshToken.getAccessToken();
        Token refreshToken = accessAndRefreshToken.getRefreshToken();

        return UriComponentsBuilder.fromUriString(targetUri)
                .queryParam("accessToken",accessToken.getJwtToken())
                .queryParam("accessTokenExpiryDate", formatter.format(accessToken.getExpiryDate()))
                .queryParam("refreshToken", refreshToken.getJwtToken())
                .queryParam("refreshTokenExpiryDate", formatter.format(refreshToken.getExpiryDate()))
                .build().toUriString();
    }

    protected void clearAuthenticationAttributes(HttpServletRequest request, HttpServletResponse response) {
        super.clearAuthenticationAttributes(request);
        httpCookieOAuth2AuthorizationRequestRepository.removeAuthorizationRequestCookies(request, response);
    }
}
```



#### OAuth2AuthenticationFailureHandler

> OAuth2 인증 중 오류가 발생하면 Spring Security는 Spring SecurityConfig에서 구성한 OAuth2AuthenticationFailureHandler의 onAuthenticationFailure () 메서드를 호출
>
> 쿼리 문자열에 추가 된 오류 메시지와 함께 사용자를 frontEndClient로 보냄.

##### OAuth2AuthenticationFailureHandler.class

```java
@Component
public class OAuth2AuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Autowired
    HttpCookieOAuth2AuthorizationRequestRepository httpCookieOAuth2AuthorizationRequestRepository;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        String targetUrl = CookieUtils.getCookie(request, REDIRECT_URI_PARAM_COOKIE_NAME)
                .map(Cookie::getValue)
                .orElse(("/"));

        targetUrl = UriComponentsBuilder.fromUriString(targetUrl)
                .queryParam("error", exception.getLocalizedMessage())
                .build()
                .toUriString();

        httpCookieOAuth2AuthorizationRequestRepository.removeAuthorizationRequestCookies(request, response);

        getRedirectStrategy().sendRedirect(request, response, targetUrl);
    }
}
```

