## 栈和队列

### 一. 栈
#### 1. 使用 WeakMap 实现一个栈函数
```js
const Stack = (function() {
const items = new WeakMap();

class Stack {
  constructor() {
    items.set(this, []);
  }
  // 入栈
  push(item) {
    items.get(this).push(item);
  }
  // 出栈
  pop() {
    return items.get(this).pop();
  }
  // 获取当前栈顶
  peek() {
    return items.get(this)[items.get(this).length - 1];
  }
  // 栈长度
  size() {
    return items.get(this).length;
  }
  // 栈是否为空
  isEmpty() {
    return items.get(this).length === 0;
  }
  // 清空栈
  clear() {
    items.get(this).length = 0;
  }
  getItems() {
    return items;
  }
}
return Stack;
})();

let a = new Stack();
let b = new Stack();
a.push('a');
b.push('b');
console.log(a.getItems()); // WeakMap {Stack => Array(1), Stack => Array(1)}
// 让垃圾回收机制自动清除弱引用内存
b = null;
setTimeout(() => {
console.log(a.getItems()); // WeakMap {Stack => Array(1)}
}, 10000);
```

#### 2. 利用栈数据结构实现十进制转二进制
```js
/**
 * 利用栈数据结构实现十进制转二进制 10 => 1010
 * 
 * 10 / 2 => 5...0
 * 5 / 2  => 2...1
 * 2 / 2  => 1...0
 * 1 / 2  => 0...1
 */
  const baseConverter = (number, base = 2) => {
  const items = new Stack();
  let result = '';

  while (number > 0) {
    items.push(number % base);
    number = Math.floor(number / base);
  }
  
  while (!items.isEmpty()) {
    result += items.pop();
  }

  return result;
};

baseConverter(10, 2); // 1010
```

### 二. 队列
#### 1. 使用 WeakMap 实现一个队列函数
```js
const Queue = (function() {
const items = new WeakMap();
class Queue {
  constructor() {
    items.set(this, []);
  }
  // 入列
  enqueue(item) {
    items.get(this).push(item);
  }
  // 出列
  dequeue() {
    return items.get(this).shift();
  }
  // 获取当前队列首位
  front() {
    return items.get(this)[0];
  }
  // 栈长度
  size() {
    return items.get(this).length;
  }
  // 栈是否为空
  isEmpty() {
    return items.get(this).length === 0;
  }
  // 清空队列
  clear() {
    items.get(this).length = 0;
  }
  getItems() {
    return items;
  }
}
return Queue;
})();
```

#### 2. 使用普通队列实现击鼓传花游戏函数
```js
const hotPotato = function (list) {
const queue = new Queue();
list.forEach(name => queue.enqueue(name));

return function (number) {
  if (queue.isEmpty()) {
    console.log('游戏已经结束');
    return;
  }

  for (let i = 0; i < number; i++) {
    queue.enqueue(queue.dequeue());
  }
  const outer = queue.dequeue();
  console.log(`${outer}出局了`);

  if (queue.size() === 1) {
    const winner = queue.dequeue();
    queue = null;
    console.log(`${winner}获胜`);
    return winner;
  } else {
    return outer;
  }
  console.log(number, queue.getItems());
};
};

const game = hotPotato(['鼠', '牛', '虎', '兔', '龙', '蛇', '马', '羊', '猴', '鸡', '狗', '猪']);
game(12); // 鼠出局了
game(22); // 牛出局了
// ...
game(32); // 龙获胜
```