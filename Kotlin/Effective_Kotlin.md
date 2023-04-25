# EffectiveKotlin 



### 아이템 1 : 가변성을 제한하라 

1. Type Inferred (x) -> fun / val / var : Type 명시

   ```kotlin
   val map: MutableMap<Int, String> = mutableMapOf()
   val map = mutableMapOf<Int,String>()
   ```



### 아이템 3 : 최대한 플랫폼 타입을 사용하지 마라

1. !! 을 지양하라 

```kotlin
data!! ? // 잘못된 코드

// ex 1)
val data = data ?: return@forEach

// ex 2)
if (data != null) {}

// ex 3)
data?.let { data -> }
```

2. typealias를 지양하라

   

### 아이템 11 : 가독성을 목표로 설계하라 

```kotlin
// 좋은 예
if (person != null && person.isAdult) {
  view.showPerson(person)
} else {
  view.showError()
}

// 나쁜 예
person?.takeIf { it.isAdult }
	?.let(view::showPerson)
	?: view.showError()
```

