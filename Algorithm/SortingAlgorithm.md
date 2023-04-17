## 정렬 알고리즘

#### 1. 버블정렬(Bubble Sort)

> 버블정렬이란? 배열의 두 수 [a, b]를 비교하여 크기가 순서대로 정리되어있지 않다면 자리를 교환하여 정렬하는 방식
>
> 시간복잡도 : O(N^2)
> 공간복잡도 : O(N)

구현 코드 : `kotlin`

```kotlin
/**
 * 버블정렬
 * IntArray 확장함수 사용
 */
internal fun IntArray.bubbleSort(): IntArray {
    // 캡처링
    val array = this

    for (i in array.indices) {
        for (j in 0 until array.size - 1) {
            /**
             * 오름차순 정렬
             * 내림차순이면 조건을 array[j] < array[j + 1]
             */
            if (array[j] > array[j + 1]) {
                val temp = array[j]
                array[j] = array[j + 1]
                array[j + 1] = temp
            }
        }
    }
    return array
}

fun main() {
    val array = intArrayOf(5, 3, 7, 9, 1)

    print(array.bubbleSort().toList())
}
```



#### 2. 선택정렬(Selection Sort)

> 선택정렬이란? 배열의 최솟값을 찾아 가장 앞의 데이터와 교환하여 정렬하는 방식
>
> 시간복잡도 : O(N^2)
> 공간복잡도 : O(N)

구현 코드 : `kotlin`

```kotlin
/**
 * 선택정렬
 * IntArray 확장함수 사용
 */
internal fun IntArray.selectionSort(): IntArray {
    // 캡처링
    val array = this

    // 최솟값
    var min:Int = 0

    for (i in array.indices) {
         min = i

        /**
         * 오름차순 정렬
         * 내림차순 이면 조건을 array[min] < array[j]
         */
        for (j in i + 1 until array.size) {
            // 만약 min 번째 보다 j 번째 값이 작다
            if (array[min] > array[j]) {
                min = j
            }
        }

        // 가장 낮은값 와 현재값 스왑
        val temp = array[min]
        array[min] = array[i]
        array[i] = temp
    }
    return array
}

fun main() {
    val array = intArrayOf(5, 3, 7, 9, 1)

    print(array.selectionSort().toList())
}
```



#### 3. 삽입정렬(Insertion Sort)

> 삽입정렬이란? 모든 요소를 앞에서부터 차례대로 이미 정렬된 배열 부분과 비교하여, 자신의 위치를 찾아 삽입하는 방식
>
> 시간복잡도 : O(N^2) = 배열이 역순으로 정렬된 경우 or O(N) = 배열이 이미 정렬되어 있는 상태일때
> 공간복잡도 : O(N)

구현 코드 : `kotlin`

```	kotlin
/**
 * 삽입정렬
 * IntArray 확장함수 사용
 */
internal fun IntArray.insertionSort(): IntArray {
    // 캡처링
    val array = this

    for (i in 1 until array.size) {
        // 현재값
        val current = array[i]
        var j = i - 1

        /**
         * 오름차순 정렬
         * 내림차순 이면 조건을 j >= 0 && array[j] < current
         */
        while (j >= 0 && array[j] > current) {
            // 배열 오른쪽으로 이동
            array[j + 1] = array[j--]
        }
        // 현재값 저장
        array[j + 1] = current
    }
    return array
}

fun main() {
    val array = intArrayOf(5, 3, 7, 9, 1)

    print(array.insertionSort().toList())
}
```



#### 4. 병합정렬(Merge Sort)

> 병합정렬이란? 분할 정복 방식을 이용해 배열을 반으로 더 이상 나눌 수 없을 때 까지 반으로 나누고, 나누어진 부분을 두 개씩 짝지어 정렬하며 합치는 과정을 반복 하는 방식
>
> 시간복잡도 : O(N*logN)
> 공간복잡도 : O(N)

구현 코드 : `kotlin`

```kotlin
/**
 * 합병정렬
 * IntArray 확장함수 사용
 */
internal fun IntArray.mergeSort(): IntArray {
    val arr = this
    if (arr.size < 2) {
        return arr
    }

    val left: IntArray = arr.copyOfRange(0, arr.size / 2)
    val right: IntArray = arr.copyOfRange(arr.size / 2, arr.size)

    return merge(arr, left.mergeSort(), right.mergeSort())
}

fun merge(arr: IntArray, left: IntArray, right: IntArray): IntArray {
    var leftIndex: Int = 0
    var rightIndex: Int = 0
    var index: Int = 0

    // 배열에 앞부분부터 왼쪽 오른쪽 비교
    while (leftIndex < left.size && rightIndex < right.size) {
        val leftCurrent = left[leftIndex]
        val rightCurrent = right[rightIndex]

        /**
         * 오름차순 정렬
         * 내림차순 이면 조건을 leftCurrent > rightCurrent
         */
        if (leftCurrent < rightCurrent) {
            arr[index] = left[leftIndex]
            leftIndex++
        } else {
            arr[index] = right[rightIndex]
            rightIndex++
        }
        index++
    }

    while (leftIndex < left.size) {
        arr[index] = left[leftIndex]
        leftIndex++
        index++
    }

    while (rightIndex < right.size) {
        arr[index] = right[rightIndex]
        rightIndex++
        index++
    }

    return arr
}

fun main() {
    val array = intArrayOf(5, 3, 7, 9, 1)

    println(array.mergeSort().toList())
}
```



#### 5. 퀵 정렬(Quick Sort)

> 퀵 정렬이란? 분할 정복 방법을 통해 배열을 정렬, 배열 가운데 하나의 값을 고르는데 이것을 피벗이라고 하고 피벗보다 값이 작은 모든 값들은 앞으로 피벗보다 큰 값들은 뒤로
> 이렇게 피벗을 기준으로 배열을 나누고 합치는 과정을 반복 하는 방식
>
> 시간복잡도 : O(N*logN) = 최선의 경우 O(N^2) = 배열이 오름차순 정렬 or 내림차순 정렬되어 있을때
> 공간복잡도 : O(N)

구현 코드 : `kotlin`

```kotlin
/**
 * 퀵 정렬
 * IntArray 확장함수 사용
 */
internal fun IntArray.quickSort(): IntArray {
    // 캡처링
    val arr = this

    if (arr.size < 2) return arr

    // 피벗값
    val pivot = arr[arr.size / 2]
    // 중간 위치
    val equal = arr.filter { it == pivot }.toIntArray()
    // 피봇값 보다 작은
    val left = arr.filter { it < pivot }.toIntArray()
    // 피봇값 보다 큰
    val right = arr.filter { it > pivot }.toIntArray()

    return left.quickSort() + equal + right.quickSort()
}

fun main() {
    val array = intArrayOf(5, 3, 7, 9, 1)

    println(array.quickSort().toList())
}

```



#### 번외 : Queue Using Two Stack

구현 코드 : `kotlin`

```kotlin
import java.util.*

class MyQueue() {
    private val inputStack = Stack<Int>()
    private val outputStack = Stack<Int>()

    /**
     * 값 x를 스택에 저장
     */
    fun push(x: Int) {
        inputStack.push(x)
    }

    /**
     * 큐 앞의 값을 제거하고 해당 값 반환
     */
    fun pop(): Int {
        peek()
        return outputStack.pop()
    }

    /**
     * 큐 앞의 값을 가져옴
     */
    fun peek(): Int {
        if (outputStack.isEmpty()) {
            while (inputStack.isNotEmpty()) {
                outputStack.push(inputStack.pop())
            }
        }
        return outputStack.peek()
    }

    /**
     * 큐가 비어 있는지 확인
     */
    fun empty(): Boolean {
        return inputStack.isEmpty() && outputStack.isEmpty()
    }

}

fun main() {
    val queue = MyQueue()
    queue.push(1)
    queue.push(2)
    println(queue.peek())
    println(queue.pop())
    print(queue.empty())
}

```

