## 二叉树的中序遍历

![二叉树的中序遍历](http://img.vanilla.ink/me/webproject/FE-Summary/Algorithm/DataStructure/03.png?x-oss-process=image/resize,w_600)

### 一. 题目
给定一个二叉树，返回它的中序遍历
```js
输入: [1, null, 2, 3]
    1
     \
      2
     /
    3
输出: [1, 3, 2]
```

### 二. 代码
#### 1. 递归实现
```js
var inorderTraversal = function (node, array = []) {
    if (node) {
        inorderTraversal(node.left, array)
        array.push(node.data)
        inorderTraversal(node.right, array)
    }
    return array
}
```

#### 2. 迭代实现
1. 取根节点为目标节点，开始遍历
2. 左子节点入栈 -> 至左子节点为空
3. 节点出栈 -> 访问该节点
4. 以右子节点为目标节点，重复1,2,3步骤
```js
var inorderTraversal = function (node) {
    var result = []
    var stack = []
    var current = node
    while (current || stack.length > 0) {
        while (current) {
            stack.push(current)
            current = current.left
        }
        current = stack.pop()
        result.push(current.data)
        current = current.right
    }
    return result
}
```
