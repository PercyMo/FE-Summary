## 链表

### 一. 单向链表
```js
const LinkedList = (function () {
  const Node = function (element) {
    this.element = element;
    this.next = null;
  };

  class LinkedList {
    constructor () {
      this.head = null;
      this.length = 0;
    }
    append (element) {
      const newNode = new Node(element);

      if (!this.head) {
        // 没有头结点，将新节点设置为头结点
        this.head = newNode;
      } else {
        let current = this.head;
        // 遍历找到链表尾部
        while (current.next) {
          current = current.next;
        }
        current.next = newNode;
      }

      this.length++;
    }
    insert (position, element) {
      if (position < 0 || position > this.length) {
        return false;
      }

      const newNode = new Node(element);
      
      if (position === 0) {
        // 往链表首部添加新节点
        newNode.next = this.head;
        this.head = newNode;
      } else {
        // 非链表首部添加新节点
        let index = 0;
        let current = this.head;
        let previous;
        while (index++ < position) {
          previous = current;
          current = current.next;
        }
        newNode.next = current;
        previous.next = newNode;
      }
      this.length++;
      return true;
    }
    removeAt (position) {
      if (position < 0 || position > this.length || !this.length) {
        return null;
      }

      let current = this.head;
      if (position === 0) {
        this.head = current.next;
      } else {
        let index = 0;
        let previous = null;
        while (index++ < position) {
          previous = current;
          current = current.next;
        }
        previous.next = current.next;
      }

      this.length--;
      return current.element;
    }
    // 将链表的值字符串化
    toString (symbol) {
      let current = this.head;
      let result = '';

      while (current) {
        result += current.element;
        current = current.next;
        if (current) {
          result += symbol ? symbol : ',';
        }
      }

      return result;
    }
    indexOf (element) {
      let index = 0;
      let current = this.head;
      while (current) {
        if (current.element === element) {
          return index;
        }
        current = current.next;
        index++;
      }
      return -1;
    }
    find (element) {
      let current = this.head;
      while (current) {
        if (current.element === element) {
          return current;
        }
        current = current.next;
      }
    }
    getHead () {
      return this.head;
    }
    size () {
      return this.length;
    }
    isEmpty () {
      return this.length === 0;
    }
  };

  return LinkedList;
})();
```

### 二. 双向链表
```js
  const DoubleLinkedList = (function () {
  const Node = function (element) {
    this.element = element;
    this.prev = this.next = null;
  };

  class DoubleLinkedList {
    constructor () {
      this.head = this.tail = null;
      this.length = 0;
    }
    append (element) {
      const newNode = new Node(element);

      if (!this.head) {
        // 没有头结点，将新节点设置为头结点
        this.head = this.tail = newNode;
      } else {
        let current = this.head;
        // 遍历找到链表尾部
        while (current.next) {
          current = current.next;
        }
        newNode.prev = current;
        current.next = this.tail = newNode;
      }

      this.length++;
    }
    insert (position, element) {
      if (position < 0 || position > this.length) {
        return false;
      }

      const newNode = new Node(element);
      
      if (position === 0) {
        // 往链表首部添加新节点
        if (!this.head) {
          this.head = this.tail = newNode;
        } else {
          newNode.next = this.head;
          this.head.prev = newNode;
          this.head = newNode;
        }
      } else if (position === this.length) {
        newNode.prev = this.tail;
        this.tail.next = newNode;
        this.tail = newNode;
      } else {
        // 非链表首部添加新节点
        let index = 0;
        let current = this.head;
        let previous;
        while (index++ < position) {
          previous = current;
          current = current.next;
        }
        previous.next = newNode;
        newNode.prev = previous;
        newNode.next = current;
        current.prev = newNode;
      }
      this.length++;
      return true;
    }
    removeAt (position) {
      if (position < 0 || position >= this.length || !this.length) {
        return null;
      }

      let current = this.head;
      if (position === 0) {
        this.head = current.next;
        if (current.next) {
          this.head.prev = null;
        } else {
          this.tail = null;
        }
      } else if (position === this.length - 1) {
        current = this.tail;
        this.tail = current.prev;
        this.tail.next = null;
      } else {
        let index = 0;
        let previous = null;
        while (index++ < position) {
          previous = current;
          current = current.next;
        }
        previous.next = current.next;
        current.next.prev = previous;
      }

      this.length--;
      return current.element;
    }
    // 删除指定节点
    removeByNode (node) {
      if (this.length === 1) {
        this.head = this.tail = null;
      } else {
        if (!node.prev) {
          this.head = node.next;
          this.head.prev = null;
        } else if (!node.next) {
          this.tail = node.prev;
          this.tail.next = null;
        } else {
          const prev = node.prev;
          const next = node.next;
          prev.next = next;
          next.prev = prev;
        }
      }
      this.length--;
    }
  };

  return DoubleLinkedList;
})();
```

### 三. 反转链表
假如有单向链表1 -> 2 -> 3 -> 4 -> 9，反转输出 9 -> 4 -> 3 -> 2 -> 1
```js
/**
 * 单向链表反转
 * -------
 * 假如有链表
 * 1 -> 2 -> 3 -> 4 -> 9
 * 
 * 完整转换过程
 * 2 -> 1 -> 3 -> 4 -> 9
 * 3 -> 2 -> 1 -> 4 -> 9
 * 4 -> 3 -> 2 -> 1 -> 9
 * 9 -> 4 -> 3 -> 2 -> 1
 */
function reverseLinkedList(head) {
  // 当前要移动的节点
  let current = null;
  // 新链表的头部节点
  let headNode = head;
  // head 节点始终为旧链表的头部节点，此处为1
  while (head && head.next) {
    // 选取 head 的下一个节点为当前要移动的节点
    current = head.next;
    // 断开 head 与 current
    head.next = current.next;
    // 将 current 移动至新链表头部
    current.next = headNode;
    // 更新 headNode 变量为新链表头部节点
    headNode = current;
  }
  return headNode;
}
```
