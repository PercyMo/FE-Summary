## 二叉树的基本操作

### 一. 完全二叉树的一些公式
1. 第n层的节点数最多为2<sup>n</sup>个节点
2. n层二叉树最多有2<sup>0</sup>+...2<sup>n</sup>=2<sup>n+1</sup>-1个节点
3. 第一个非子叶节点：length/2 // TODO: 没懂什么意思
4. 一个节点的孩子节点：2n、2n+1 // TODO: 没懂什么意思
// TODO: 完全二叉树和普通的二叉树

### 二. 基本操作
插入、遍历、深度、删除(其中删除操作相对复杂)。  

![二叉树删除节点](http://img.vanilla.ink/me/webproject/FE-Summary/Algorithm/DataStructure/01.png?x-oss-process=image/resize,w_600)

```js
class Node {
  constructor(data, left, right) {
    this.data = data;
    this.left = left;
    this.right = right;
  }
  show() {
    console.log(this.data);
  }
};

class Tree {
  constructor() {
    this.root = null;
  }
  insert(data) {
    const node = new Node(data, null, null);
    if (!this.root) {
      this.root = node;
      return;
    }
    let current = this.root;
    let parent = null;
    while (current) {
      parent = current;
      if (data < current.data) {
        current = parent.left;
        if (!current) {
          parent.left = node;
          return;
        }
      } else {
        current = parent.right;
        if (!current) {
          parent.right = node;
          return;
        }
      }
    }
  }
  // 前序遍历
  preOrder(node) {
    if (node) {
      node.show();
      this.preOrder(node.left);
      this.preOrder(node.right);
    }
  }
  // 中序遍历
  middleOrder(node) {
    if (node) {
      this.middleOrder(node.left);
      node.show();
      this.middleOrder(node.right);
    }
  }
  // 后序遍历
  laterOrder(node) {
    if (node) {
      this.laterOrder(node.left);
      this.laterOrder(node.right);
      node.show();
    }
  }
  getMin(node) {
    let current = node || this.root;
    while (current) {
      if (!current.left) {
        return current.data;
      }
      current = current.left;
    }
  }
  getMinNode(node) {
    let current = node || this.root;
    while (current) {
      if (!current.left) {
        return current;
      }
      current = current.left;
    }
  }
  getMax(node) {
    let current = node || this.root;
    while (current) {
      if (!current.right) {
        return current.data;
      }
      current = current.right;
    }
  }
  // 二叉树的最大深度
  getDeep(node, deep = 0) {
    if (!node) {
      return deep;
    }
    deep++;
    const deepLeft = this.getDeep(node.left, deep);
    const deepRight = this.getDeep(node.right, deep);
    return Math.max(deepLeft, deepRight);
  }
  // 移除节点，并返回传入的节点
  removeNode(node, data) {
    if (node === null) {
      return null;
    }

    if (data < node.data) {
      node.left = this.removeNode(node.left, data);
      return node;
    } else if (data > node.data) {
      node.right = this.removeNode(node.right, data);
      return node;
    } else {
      // 移除叶子节点
      if (!node.left && !node.right) {
        return null;
      }

      // 移除只有一个子节点的节点
      if (!node.right) {
        return node.left;
      }
      if (!node.left) {
        return node.right;
      }

      // 移除两个子节点的节点
      if (node.left && node.right) {
        // 找到右侧树中最小的节点
        const minNode = this.getMinNode(node.right);
        // 将node的值修改为最小节点的值，模拟达到节点移动效果，但此时有两个一样的节点
        node.data = minNode.data;
        // 移除右侧树上的重复节点
        this.removeNode(node.right, minNode.data);
        return node;
      }
    }
  }
}

var t = new Tree();
t.insert(3);
t.insert(8);
t.insert(1);
t.insert(2);
t.insert(5);
t.insert(7);
t.insert(6);
t.insert(0);
console.log(t);
// t.middleOrder(t.root);
console.log(t.getMin(), t.getMax());
console.log(t.getDeep(t.root, 0));
console.log(t.getNode(5,t.root));
```

### 三. 树查找
利用二分的思想
```js
getNode: function(node, data) {
  if (node) {
    if (data === node.data) {
      return node
    } else if (data < node.data) {
      return this.getNode(data, node.left)
    } else {
      return this.getNode(data, node.right)
    }
  } else {
    return null
  }
}
```
