## Spring Social-Login 정리



#### config -> AppProperties 바인딩 방법

> @ConfigurationProperties 란?
>
> *.properties, *.yml 파일에 있는 property를 자바 클래스에 값을 가져와서 사용할 수 있게 해주는 어노테이션이다.
>
> @ConfigurationProperties 를 사용해 app 으로 시작하는 propertyfmf POJO class로 바인딩

##### AppProperties.class

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



#### JWT TokenProvider

> JSON 웹 토큰을 생성하고 확인하는 코드를 만듬
>
> .claim : Token에 담을 정보
> .setSubject : Token 제목
> .setIssuedAt : 토큰 발급 시간
> .setExpiration : 토큰 만료 시간
> .signWith : 사용할 알고리즘과, 시크릿값 셋팅
> .compact : 토큰 생성

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
     * @param authToken
     */
    public void validateToken(String authToken) {
        String jwtSecret = Base64Utils.encodeToString(appProperties.getAuth().getTokenSecret().getBytes());
        Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(authToken);
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

