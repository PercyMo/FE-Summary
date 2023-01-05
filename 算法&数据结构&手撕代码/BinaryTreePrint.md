## 从上到下打印二叉树

给定二叉树: `[3,9,20,null,null,15,7]`
```js
    3
   / \
  9  20
    /  \
   15   7
```

### 一. 不分行从上到下打印
从上往下打印出二叉树的每个节点，同层节点从左至右打印。

```js
const Print = function(root) {
  if (!root) {
    return [];
  }
  const result = [];
  const queue = [];
  queue.push(root);
  while (queue.length) {
    const item = queue.shift();
    result.push(item.data);
    if (item.left) {
      queue.push(item.left);
    }
    if (item.right) {
      queue.push(item.right);
    }
  }
  return result;
};
// [3, 9, 20, 15, 7]
```

### 二. 把二叉树打印成多行
从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。

```js
const Print = function(root) {
  if (!root) {
    return [];
  }
  const result = [];
  const queue = [];
  queue.push(root);
  let temp = [];
  let leftNums = 1; // 当前层中剩下的节点数量
  let childrenNums = 0; // 下一层中子节点的数量
  while (queue.length) {
    const current = queue.shift();
    leftNums--;
    temp.push(current.data);
    if (current.left) {
      childrenNums++;
      queue.push(current.left);
    }
    if (current.right) {
      childrenNums++;
      queue.push(current.right);
    }
    if (leftNums === 0) {
      leftNums = childrenNums;
      childrenNums = 0;
      result.push(temp);
      temp = [];
    }
  }
  return result;
};
// [
//   [3],
//   [9,20],
//   [15,7]
// ]
```

### 三. 按之字形顺序打印二叉树
请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。

```js
const Print = function(root) {
  if (!root) {
    return [];
  }
  const result = [];
  const oddStack = [];
  const evenStack = [];
  let temp = [];
  oddStack.push(root);
  while (oddStack.length || evenStack.length) {
    while (oddStack.length) {
      const current = oddStack.pop();
      temp.push(current.val);
      if (current.left) {
        evenStack.push(current.left);
      }
      if (current.right) {
        evenStack.push(current.right);
      }
    }
    if (temp.length) {
      result.push(temp);
      temp = [];
    }
    while (evenStack.length) {
      const current = evenStack.pop();
      temp.push(current.val);
      if (current.right) {
        oddStack.push(current.right);
      }
      if (current.left) {
        oddStack.push(current.left);
      }
    }
    if (temp.length) {
      result.push(temp);
      temp = [];
    }
  }
  return result;
};
// [
//   [3],
//   [20,9],
//   [15,7]
// ]
```
