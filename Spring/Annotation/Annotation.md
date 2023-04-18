## LomBok 어노테이션



##### Lombok @Getter @Setter를 사용하기전 왜 Getter, Setter를 사용하는지?

> Getter/Setter를 이용해 데이터를 접근하는건 데이터에 무결성을 지키기 위해서다.
>
> 따라서 필드는 private 접근 제한자로 막고, Getter, Setter로 접근하는 방식을 사용 (코드 은닉화)



#### @Getter

> 해당 필드에 대한 기본 getMethod 생성

```java

/**
 * 순수 자바 코드 예시
 **/
public class Member {
  
  private String name;
  
  private String email;
  
  public String getName() {
    return name;
  }
  
  public String getEmail() {
    return email;
  }
}


/**
 * 롬복 사용 예시
 **/
@Getter
public class Member {
  
  private String name;
  
  private String email;
}

```



#### @Setter

> 해당 필드에 대한 기본 setMethod 생성
>
> Tip : Setter는 실무에서 지양한다. 
>
> 이유
>
> 1. 객체의 일관성을 유지하기 어렵다
> 2. 사용 의도 및 목적이 분명치 않다
>
> 이러한 이유로 변경 포인트가 Setter로 되어있고 코드가 많아지면 유지보수가 어려워서 지양하는 편이다.

```java
/**
 * 순수 자바 코드 예시
 **/
public class Member {
  
  private String name;
  
  private String email;
  
  public void setName(String name) {
      this.name = name;
  }
  public void setEmail(String email) {
      this.email = email;
  }
}

/**
 * 롬복 사용 예시
 **/
@Setter
public class Member {
  
  private String name;
  
  private String email;
}

```



#### @NoArgsConstructor

> 해당 클래스에 파라미터가 없는 기본 생성자를 만들어준다.
>
> @NoArgsConstructor 만 선언 시  기본 AccessLevel.PUBLIC과 동일하다
>
> @NoArgsConstructor(AccessLevel.PUBLIC) : 모든 Package, Class 접근 가능
>
> @NoArgsConstructor(AccessLevel.PRIVATE) : 동일 Package, 다른 Package 접근 불가 같은 Class 만 접근가능
>
> @NoArgsConstructor(AccessLevel.PROTECTED) : 다른 Package의 다른 Class 접근 불가 

```java
/**
 * 순수 자바 코드 예시
 **/
public class Member {
  
  private String name;
  
  private String email;
  
  public Member(String name, String email) {
    this.name = name;
    this.email = email;
  }
  
  public Member() {}
  
}



/**
 * 롬복 사용 예시
 **/
@NoArgsConstructor
public class Member {
  
  private String name;
  
  private String email;
  
  public Member(String name, String email) {
    this.name = name;
    this.email = email;
  }
}
```



### @AllArgsConstructor

> 모든 필드 값을 파라미터로 받는 생성자들 자동으로 만들어준다.

```java
/**
 * 순수 자바 코드 예시
 **/
public class Member {
  
  private String name;
  
  private String email;
  
  public Member(String name, String email) {
    this.name = name;
    this.email = email;
  }
  
}



/**
 * 롬복 사용 예시
 **/
@AllArgsConstructor
public class Member {
  
  private String name;
  
  private String email;
  
}
```

