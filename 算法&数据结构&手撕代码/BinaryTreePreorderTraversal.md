## 二叉树的前序遍历

![二叉树的前序遍历](http://img.vanilla.ink/me/webproject/FE-Summary/Algorithm/DataStructure/02.png?x-oss-process=image/resize,w_600)

### 一. 题目
给定一个二叉树，返回它的前序遍历
```js
输入: [1, null, 2, 3]
    1
     \
      2
     /
    3
输出: [1, 2, 3]
```

### 二. 代码
#### 1. 递归实现
```js
var preorderTraversal = function (node, array = []) {
  if (node) {
    array.push(node.data);
    preorderTraversal(node.left, array);
    preorderTraversal(node.right, array);
  }
  return array;
};
```

#### 2. 迭代实现
1. 取根节点为目标节点，开始遍历
2. 左子节点入栈，访问该节点 -> 至左子节点为空
3. 节点出栈
4. 以右子节点为目标节点，重复1,2,3步骤
```js
var preorderTraversal = function (node) {
  var result = [];
  var stack = [];
  var current = node;
  while (current || stack.length > 0) {
    while (current) {
      result.push(current.data);
      stack.push(current);
      current = current.left;
    }
    current = stack.pop();
    current = current.right;
  }
  return result;
};
```
