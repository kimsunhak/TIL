##  Lamda

**람다함수란?**
람다 함수는 프로그래밍 언어에서 사용되는 개념으로 **익명 함수(Anonymous functions)** 를 지칭하는 용어

람다 표현식

1. 람다는 매개변수 화살표 함수몸체로 이용하여 사용 할 수 있음
2. 매개변수가 하나일 경우 매개변수를 생략 가능
3. 함수몸체가 단일 실행문이면 괄호{}를 생략 가능
4. 함수몸체가 return 으로만 구성되어 있으면 괄호{}를 생략 가능

```java
// 람다 사용 x
Person person = new Person();

person.hello(new Hi() {
	public String message(String name) {
		System.out.println("안녕");
		System.out.println("반가워");
		System.out.println("내 이름은 : " + msg);
		return "String";
	}
});

// 람다 사용 O
Person person = new Person();

person.hello((name) -> {
	System.out.println("안녕");
	System.out.println("반가워");
	System.out.println("내 이름은 : " + msg);

	return "String";
});
```
