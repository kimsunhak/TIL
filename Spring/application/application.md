## Application 작성 (YML)



#### 1. YAML 이란?

> ```
> YAML(YML Ain't MarkUp Language)
> - 문법은 상대적으로 이해하기 쉽고, 가독성이 좋도록 디자인되었으며, 고급 컴퓨터 언어에
> 	적합하다
> - 사람이 쉽게 읽을 수 있는 데이터 직렬화 양식이다
> ```



#### 2. YAML 작성방법

##### application.yml

```yaml
spring:
  # DataBase 설정
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://<서버주소:port>
    username: <사용자 ID>
    password: <사용자 Password>

  # JPA Hibernate 설정
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MariaDB102Dialect
#        show_sql: true
        format_sql: true

  # 자동 빌드
  devtools:
    livereload:
      enabled: true

# Logging 설정
logging.level:
    org.hibernate.SQL: debug
#    org.hibernate.type: trace


# Server Port 설정
server:
  port: 8080
```

> ##### Hibernate ddl-auto 정리
>
> 1. create-drop : SessionFactory가 시작될 때 drop 및 생성을 실행하고, SessionFactory가 종료될 때 drop을 실행
> 2. create : SessionFactory가 시작될 때 drop을 실행하고 DDL을 실행
> 3. update : 변경된 Schema를 적용
> 4. validate: 변경된 Schema가 있으면 변경된 점을 출력하고 Application을 종료

> ##### Dialect (방언) 설정
>
> Dialect는 Hibernate 에서 다양한 데이터베이스 문법을 처리하기 위해 각 데이터베이스에 맞는 SQL문법을 처리하기 위해 사용한다
>
> 문법으로는 : 
>
> 1. org.hibernate.dialect.MariaDB102Dialect => MariaDB
> 2. org.hibernate.dialect.MySQL5Dialect => MySQL DB
> 3. org.hibernate.dialect.PostgreSQLDialect => PostgreSQL DB

