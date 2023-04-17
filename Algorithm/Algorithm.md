# 알고리즘 정리

### 1. 시간 복잡도 

> 문제를 해결하기 위한 시간(연산)의 횟수를 말함
>
> 시간 복잡도를 나타날때는 Big O 표기법을 사용

#### 예제 : 1 ~ N 까지의 합을 구한다고 할때

##### 시간복잡도 O(n) 

```kotlin
/**
 * 시간복잡도 알고리즘 1번
 * for 문을 사용하여 N번의 연산을 해서 시간복잡도로 표현하면 O(N)
 */
fun sumAll(num: Int): Int {
    var total:Int = 0

    for (i in 0 .. num) {
        total += i
    }

    return total
}

fun main() {
    print(sumAll(100))
}
```

##### 시간복잡도 O(1) 

```kotlin
/**
 * 시간복잡도 알고리즘 2번
 * 일반적인 수식으로 표현해서 n이 몇번이든 1번의 연산을하여 시간복잡도는 O(1)
 */
fun sumAll(num: Int): Int {
    return (num * (num + 1) / 2)
}

fun main() {
    print(sumAll(100))
}
```



