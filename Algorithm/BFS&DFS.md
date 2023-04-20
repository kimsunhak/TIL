## BFS(너비 우선 탐색) & DFS (깊이 우선 탐색)

#### 1. BFS (너비 우선 탐색)

> 특정 노드부터 시작해 가까운 노드부터 탐색하는 방식
>
> - 특징
    >
- 시작 노드의 가장 가까운 노드부터 가장 먼 노드 순으로 모든 노드 탐색
>   - 노드의 수가 적고 깊이가 얕은 경우 `빠르게 동작`
>   - 거리순으로 노드를 탐색, `최단경로` 가 존재한다면 반드시 찾음
>   - 큐를 활용해서 탐색할 노드의 순서를 저장하고 큐에 저장된 순서대로 탐색 - 노드의 수가 많을 수록 더 큰 저장공간이 필요
> - 원리
    >
- 시작 노드를 탐색, 인접 노드를 `Queue`에 삽입
>   - Loop : while(queue.isNotEmpty())
>   - `Queue`의 가장 앞에 있는 노드를 꺼내 탐색
>   - 방문하지 않은 노드들을 `Queue`에 `push`


<p align="center"><img src="../assets/graph.png" alt="graph" width="300" /></p>

```kotlin
import java.util.*

/**
 * BFS - Graph
 * Array<IntArray> 확장함수 사용
 */
internal fun Array<IntArray>.bfs(start: Int, visited: BooleanArray) {
    val graph = this
    val queue = LinkedList<Int>()

    queue.offer(start) // 시작할 노드 번호
    visited[start] = true // 시작노드 방문처리

    while (queue.isNotEmpty()) {
        val x = queue.poll() // 첫번째 값 반환 후 remove
        println("x : $x")

        for (y in graph[x]) {
            if (!visited[y]) {
                queue.offer(y)
                visited[y] = true
            }
        }
    }
}

fun main() {
    val graph = arrayOf(
        intArrayOf(1),
        intArrayOf(2, 3, 4),
        intArrayOf(1, 5, 6),
        intArrayOf(1),
        intArrayOf(1, 7),
        intArrayOf(2, 8),
        intArrayOf(2, 8),
        intArrayOf(4),
        intArrayOf(5, 6),
    )

    val visited = BooleanArray(graph.size)
    graph.bfs(0, visited)
}
```

```kotlin
import java.util.*

/**
 * BFS - TreeNode
 */
class Node<T>(
    var value: T,
    var left: Node<T>? = null,
    var right: Node<T>? = null,
    var visited: Boolean = false
)

class BreathFirstSearch<T>(
    var root: Node<T>? = null,
) {
    fun bfs(root: Node<T>): MutableList<List<T>> {
        val queue = LinkedList<Node<T>>()
        val tree = mutableListOf<List<T>>()

        queue.offer(root) // 시작노드
        root.visited = true // 시작노드 방문처리

        while (queue.isNotEmpty()) {
            val container = mutableListOf<T>()

            for (i in 0 until queue.size) { // Queue에 들어있는 root
                val node = queue.poll() // 현재 node

                container.add(node.value) // 현재 노드값 container에 저장

                // 노드에 왼쪽 자식 노드가 존재하는경우
                node.left?.let { leftNode ->
                    if (!leftNode.visited) {
                        queue.offer(leftNode)
                        leftNode.visited = true
                    }
                }
                // 노드에 오른쪽 자식 노드가 존재하는경우
                node.right?.let { rightNode ->
                    if (!rightNode.visited) {
                        queue.offer(rightNode)
                        rightNode.visited = true
                    }
                }
            }
            tree.add(container)
        }
        return tree
    }
}

fun main() {
    val node: Node<Int> = Node(3, Node(9), Node(20, Node(15), Node(7)))
    val bfs: BreathFirstSearch<Int> = BreathFirstSearch()

    print(bfs.bfs(node))
}
```
```kotlin
import java.util.*

/**
 * 4방향 순회
 */
enum class Direction(val x: Int, val y: Int) {
    Up(-1, 0),
    Right(0, 1),
    Down(1, 0),
    Left(0, -1)
}

/**
 * BFS - Grid
 */
fun bfs(grid: Array<IntArray>, start: Pair<Int, Int>, dest: Pair<Int, Int>): Int {

    val queue = LinkedList<Pair<Int, Int>>()
    val visited = mutableSetOf<String>()

    var distance = 0

    queue.add(start)
    visited.add("$start.first, $start.second")

    while (queue.isNotEmpty()) {
        // 이번 distance 의 노드를 다 방문
        for (i in 0 until queue.size) {
            // 방문할 노드
            val (x, y) = queue.poll()
            // 4방향 다 점검
            Direction.values().forEach { di ->
                val (nx, ny) = x + di.x to y + di.y
                // 방문 할 수 있는 조건
                // 1. 맵 안에 있어야함
                // 2. 길인지
                // 3. 방문했는지
                if (
                    (nx in grid.indices && ny in grid[0].indices) &&
                    grid[nx][ny] != -1 &&
                    !visited.contains("$nx, $ny")
                ) {
                    val next = nx to ny
                    // 방문지인지 찾음
                    if (dest == next) return distance + 1
                    // 아니라면 큐에 넣음 + 방문 했다고 가정하고 처리
                    queue.add(next)
                    visited.add("$nx, $ny")
                }
            }
        }
        // 내가 최대 탐색가능한 거리까지 출력하고 종료
        if (queue.isEmpty()) {
            return distance
        }
        // 시작좌표로부터 거리 + 1
        distance++
    }
    error("it can't be")
}

fun generatePos(height: Int, width: Int): Pair<Int, Int> {
    val x = (0 until height).random()
    val y = (0 until width).random()
    return x to y
}

fun main() {
    // 1 = 도착지
    // 0 = 길 ( 이동할 수 있는 길 )
    // -1 = 막다른 길 ( 갈 수 없는 길 )
    // 만약 갈 수 없는 길 => 내가 이동할 수 있는 최대거리를 반환해라
    val grid = Array(5) { IntArray(5) { listOf(0, -1).random() } }
    val start = generatePos(grid.size, grid[0].size)
    val dest: Pair<Int, Int> = run {
        var tmp = generatePos(grid.size, grid[0].size)
        while (start == tmp) tmp = generatePos(grid.size, grid[0].size)
        tmp
    }
    grid[start.first][start.second] = 0
    grid[dest.first][dest.second] = 1

    for (ints in grid) {
        println(ints.toList())
    }
    val answer = bfs(grid, start, dest)

    println("시작 좌표 : $start / 도착 좌표 : $dest")
    println("최소 이동 거리 : $answer")
}
```
