## Binary Tree

#### 이진트리 구현 및 트리 순회 (전위, 중위, 후위)

> 이진트리란? 각 노드가 최대 두 개의 자식을 갖는 트리.
> 즉, 각 노드는 자식이 존재하지 않거나 한 개 이거나 두 개 만을 갖는 것
>
> 탐색 방법 : 전위 순회, 중위 순회, 후위 순회

구현 코드 : `kotlin`

```kotlin
/**
 * 이진트리 구현,
 * 트리 순회 구현 : preorder, inorder, postorder
 */
class Node<T>(
    var value: T,
    var left: Node<T>? = null,
    var right: Node<T>? = null,
)

class BinaryTree<T>(
    var root: Node<T>? = null,
) {

    /**
     * 전위 순회 (root, leftNode, rightNode)
     */
    fun preorder(root: Node<T>) {
        print(root.value)
        if (root.left != null) preorder(checkNotNull(root.left))
        if (root.right != null) preorder(checkNotNull(root.right))
    }

    /**
     * 중위 순회 (leftNode, root, rightNode)
     */
    fun inorder(root: Node<T>) {
        if (root.left != null) inorder(checkNotNull(root.left))
        print(root.value)
        if (root.right != null) inorder(checkNotNull(root.right))
    }

    /**
     * 후위 순회 (leftNod, rightNode, root)
     */
    fun postorder(root: Node<T>) {
        if (root.left != null) postorder(checkNotNull(root.left))
        if (root.right != null) postorder(checkNotNull(root.right))
        print(root.value)
    }
}

fun main() {
    val node: Node<Char> = Node('A', Node('B', Node('D'), Node('E')), Node('C', Node('F'), Node('G')))
    val tree: BinaryTree<Char> = BinaryTree()
    print("preorder result : ")
    tree.preorder(node)
    println()
    print("inorder result : ")
    tree.inorder(node)
    println()
    print("postorder result : ")
    tree.postorder(node)
}
```

#### 이진 탐색 트리 (Binary Search Tree, BST)

> 이진 탐색 트리란? 각 노드가 최대 두개의 자식을 갖는 트리이며,
> 즉 왼쪽 자식 노드는 부모 노드 보다 값이 작고, 오른쪽 자식 노드는 부모보다 값이 크다
>
> 메서드 구현 : 검색, 삽입, 삭제
>
> 시간복잡도 : O(logN)
> 공간복잡도 : O(N)

구현 코드 : `kotlin`

```kotlin
/**
 * 이진 검색 트리 구현
 * 메서드 구현 : 검색, 삽입, 삭제
 */
class Node<T>(
    var value: T,
    var left: Node<T>? = null,
    var right: Node<T>? = null,
) {
    val min: Node<T>
        get() = left?.min ?: this
}

// T: Comparable 사용하여 비교
class BinarySearchTree<T : Comparable<T>>(
    var root: Node<T>? = null,
) {

    // Node 삽입
    fun insert(value: T) {
        root = insert(root, value)
    }

    private fun insert(root: Node<T>?, value: T): Node<T> {
        if (root == null) return Node(value)

        if (value < root.value) {
            root.left = insert(root.left, value)
        } else {
            root.right = insert(root.right, value)
        }

        return root
    }

    // Node 검색
    fun search(value: T): Boolean {
        // 햔재 rootNode
        var current = root

        while (current != null) {
            if (current.value == value) {
                return true
            }

            // 왼쪽 or 오른쪽 노드 확인
            current = if (value < current.value) current.left else current.right
        }
        return false
    }

    // Node 삭제
    fun remove(value: T) {
        root = remove(root, value)
    }

    fun remove(root: Node<T>?, value: T): Node<T>? {
        if (root == null) return null

        if (root.left == null) {
            return root.right
        }

        if (root.right == null) {
            return root.left
        }

        // 오른쪽 완쪽 둘다 있을때 -> 오른쪽 가장 작은 값과 change
        root.right?.min?.value.let {
            if (it != null) {
                root.value = it
            }
            root.right = remove(root.right, root.value)
        }

        when {
            value < root.value -> root.left = remove(root.left, value)
            else -> root.right = remove(root.right, value)
        }
        return root
    }
}

fun main() {
    val tree = BinarySearchTree<Int>().apply {
        insert(3)
        insert(4)
        insert(0)
        insert(2)
        insert(1)
        insert(5)
        insert(6)
    }

    if (tree.search(5)) println("존재 함") else println("존재 하지 않음")
    tree.remove(5)
    if (tree.search(5)) println("존재 함") else println("존재 하지 않음")

    if (tree.search(3)) println("존재 함") else println("존재 하지 않음")
    tree.remove(3)
    if (tree.search(3)) println("존재 함") else println("존재 하지 않음")
}
```





