## 路径总和

### 一.路径总和
#### 题目
给你二叉树的根节点 `root` 和一个表示目标和的整数 `targetSum` 。判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和 `targetSum` 。如果存在，返回 `true` ；否则，返回 `false` 。  
[leetcode - 112. 路径总和](https://leetcode.cn/problems/path-sum/solution/lu-jing-zong-he-by-leetcode-solution/)

#### 代码
广度优先
```js
var hasPathSum = function(root, targetSum) {
  if (!root) {
    return false;
  }
  const nodeStack = [];
  const valStack = [];
  nodeStack.push(root);
  valStack.push(root.val);
  while (nodeStack.length > 0) {
    const node = nodeStack.shift();
    const val = valStack.shift();
    if (!node.left && !node.right) {
      if (val === targetSum) {
        return true
      }
    }
    if (node.left) {
      nodeStack.push(node.left);
      valStack.push(val + node.left.val);
    }
    if (node.right) {
      nodeStack.push(node.right);
      valStack.push(val + node.right.val);
    }
  }
  return false;
};
```

深度优先
```js
var hasPathSum = function(root, targetSum) {
  if (!root) {
    return false;
  }
  if (!root.left && !root.right) {
    return targetSum === root.val;
  }
  return hasPathSum(root.left, targetSum - root.val) ||
    hasPathSum(root.right, targetSum - root.val);
};
```