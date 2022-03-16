## 二叉树的后序遍历

![二叉树的后序遍历](http://img.vanilla.ink/me/webproject/FE-Summary/Algorithm/DataStructure/04.png?x-oss-process=image/resize,w_600)

### 一. 题目
给定一个二叉树，返回它的后序遍历
```js
输入: [1, null, 2, 3]
    1
     \
      2
     /
    3
输出: [3, 2, 1]
```

### 二. 代码
#### 1. 递归实现
```js
var postorderTraversal = function (node, array = []) {
    if (node) {
        postorderTraversal(node.left, array)
        postorderTraversal(node.right, array)
        array.push(node.data)
    }
    return array
}
```

#### 2. 迭代实现
1. 取根节点为目标节点，开始遍历
2. 左子节点入栈 -> 至左子节点为空
3. 栈顶节点的右节点为空或右节点被访问过 -> 节点出栈并访问他，将节点标记为已访问
4. 栈顶节点的右节点不为空且未被访问，以右子节点为目标节点，重复1,2,3步骤
```js
var postorderTraversal = function (node) {
    var result = [];
    var stack = [];
    var current = node;
    var last = null; // 标记上一个访问的节点
    while (current || stack.length > 0) {
        while (current) {
            stack.push(current);
            current = current.left;
        }

        current = stack[stack.length - 1];
        if (!current.right || current.right === last) {
            current = stack.pop();
            result.push(current.data);
            last = current;
            current = null; // 继续弹栈
        } else {
            current = current.right;
        }
    }
    return result
}
```
