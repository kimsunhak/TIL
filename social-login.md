## Spring Social-Login 정리



#### config -> AppProperties 바인딩 방법

> @ConfigurationProperties 란?
>
> *.properties, *.yml 파일에 있는 property를 자바 클래스에 값을 가져와서 사용할 수 있게 해주는 어노테이션이다.

##### AppProperties.class

~~~java
@ConfigurationProperties(prefix = "app")
public class AppProperties {
}
~~~

##### *.properties

```
# 예시
app.naver=https://www.naver.com
```



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

