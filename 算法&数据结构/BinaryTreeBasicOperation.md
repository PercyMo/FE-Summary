## 二叉树的基本操作

### 一. 二叉树的一些公式
1. 第n层的节点数最多为2<sup>n</sup>个节点
2. n层二叉树最多有2<sup>0</sup>+...2<sup>n</sup>=2<sup>n+1</sup>-1个节点
3. 第一个非子叶节点：length/2 // TODO: 没懂什么意思
4. 一个节点的孩子节点：2n、2n+1 // TODO: 没懂什么意思

### 二. 基本结构
插入、遍历、深度 // TODO: 缺少了preOrder、middleOrder、laterOrder方法
```js
function Node(data, left, right) {
    this.data = data
    this.left = left
    this.right = right
}

Node.prototype = {
    show: function () {
        console.log(this.data)
    }
}

function Tree() {
    this.root = null
}

Tree.prototype = {
    insert: function (data) {
        const node = new Node(data, null, null)
        if (!this.root) {
            this.root = node
            return
        }
        let current = this.root
        let parent = null
        while (current) {
            parent = current
            if (data < current.data) {
                current = parent.left
                if (!current) {
                    parent.left = node
                    return
                }
            } else {
                current = parent.right
                if (!current) {
                    parent.right = node
                    return
                }
            }
        }
    },
    getMin: function () {
        let current = this.root
        while (current) {
            if (!current.left) {
                return current.data
            }
            current = current.left
        }
    },
    getMax: function () {
        let current = this.root
        while (current) {
            if (!current.right) {
                return current.data
            }
            current = current.right
        }
    },
    getDeep: function (node, deep) {
        deep = deep || 0
        if (!node) {
            return deep
        }
        deep++
        const deepLeft = this.getDeep(node.left, deep)
        const deepRight = this.getDeep(node.right, deep)
        return Math.max(deepLeft, deepRight)
    },
}
```
```js
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
getNode: function(data, node) {
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

### 四. 二分查找
二分查找的条件是必须是有序的线性表。
和线性表的中点值进行比较，如果小就继续在小的序列中查找，如此递归直到找到相同的值。
```js
const arr = [0, 1, 1, 1, 1, 1, 4, 6, 7, 8]
function binarySearch(data, arr, start, end) {
    if (start > end) {
        return -1;
    }
    const mid = Math.floor((end + start) / 2)
    if (data === arr[mid]) {
        return mid
    } else if (data < arr[mid]) {
        return binarySearch(data, arr, start, mid - 1)
    } else {
        return binarySearch(data, arr, mid + 1, end)
    }
}
console.log(binarySearch(1, arr, 0, arr.length - 1))
```
