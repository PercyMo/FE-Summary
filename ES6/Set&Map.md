## Set、Map和WeakSet、WeakMap
`Set` 和 `Map` 的主要应用场景在于 **数据重组** 和 **数据存储**  
`Set`是一种叫做**集合**的数据结构，`Map`是一种叫做**字典**的数据结构

### 一. 集合（Set）
ES6 新增的一种数据结构，类似于数组，但成员是唯一且无序的，没有重复的值。  
**Set本身是一个构造函数，用于生成Set数据结构。**
```js
new Set([iterable])
```
例子：
```js
const s = new Set()
[1, 2, 3, 4, 3, 2, 1].forEach(v => s.add(v))

for (let i of s) {
    console.log(i)  // 1 2 3 4
}
```
Set 对象允许你存储任意类型的唯一值，无论是原始类型或是对象引用。  
向Set加入值的时候不会发生类型转换，所以`5`和`'5'`是两个不同的值。Set内部判断两个值是否相同，使用的算法叫做“Same-value-zero equality”，它类似于**精确相等运算符**（`===`），主要的区别是：  
**Set 认为`NaN`等于自身，而精确相等运算符认为`NaN`不等于自身**
```js
const set = new Set()
const a = NaN
const b = NaN
set.add(a)
set.add(b)
set     // Set {NaN}

const set1 = new Set()
set1.add(5)
set1.add('5')
set     // Set {5, "5"}
```

#### 1. 实例属性
* constructor: 构造函数
* size: 元素数量  
```js
const set = new Set([1, 2, 3, 2, 1])

console.log(set.length)     // undefined
console.log(set.size)       // 3
```

#### 2. 实例方法
* `add(value):` 新增，相当于array中的push
* `delete(value)`: 删除集合中的value
* `has(value)`: 判断集合中是否存在value
* `clear()`: 清空集合

```js
const set = new Set()
set.add(1).add(2).add(1)

set.has(1)      // true
set.has(3)      // false
set.delete(1)
set.has(1)      // false
```

`Array.from`方法可以将`Set`结构转为数组

```js
const set = new Set([1, 2, 3, 2])
const array = Array.from(set)
console.log(array)      // [1, 2, 3]
// 或者
const arr = [...set]
console.log(arr)        // [1, 2, 3]
```

#### 3. 遍历操作
* `keys():` 返回一个包含集合中所有键的迭代器
* `values()`: 返回一个包含集合中所有值的迭代器
* `entries()`: 返回一个包含Set中所有键值对的迭代器
* `forEach(callbackFn, thisArg)`: 用于对集合成员执行`callbackFn`操作，如果提供`thisArg`参数，回调中会将这个参数绑定为上下文
```js
const set = new Set([1, 2, 3])
console.log(set.keys())     // SetIterator {1, 2, 3}
console.log(set.values())   // SetIterator {1, 2, 3}
console.log(set.entries())  // SetIterator {1, 2, 3}

for (let i of set.keys()) {
    console.log(i)
}       // 1  2  3
for (let i of set.entries()) {
    console.log(i)
}       // [1, 1]  [2, 2]  [3, 3]

set.forEach((key, value) => {
    console.log(key + ' : ' + value)
})      // 1 : 1  2 : 2  3 : 3
console.log([...set])   // [1, 2, 3]
```

**Set 可默认遍历，默认迭代器生成函数是`values()`**
// TODO: 没明白什么意思，看过迭代器再回来看下
```js
Set.prototype[Symbol.iterator] === Set.prototype.values // true
```

**所以，Set可以使用`map()`、`filter()`**
```js
let set = new Set([1, 2, 3])
set = new Set([...set].map(v => v * 2))
console.log([...set])   // [2, 4, 6]

set = new Set([...set].filter(v => (v >= 4)))
console.log([...set])   // [4, 6]
```

**使用Set实现数组去重、并集、交集、差集**
```js
// 两种数组去重方法
const arr = [1, 2, 3, 2, 1, 1]
[...new Set(arr)]         // [1, 2, 3]
Array.from(new Set(arr))  // [1, 2, 3]


const a = new Set([1, 2, 3])
const b = new Set([4, 3, 2])

// 并集
const union = new Set([...a, ...b])
// Set {1, 2, 3, 4}

// 交集
const intersect = new Set([...a].filter(x => b.has(x)))
// Set {2, 3}

// 差集
const difference = new Set([...a].filter(x => !b.has(x)))
// Set {1}
```

### 二. WeakSet
WeakSet 允许将一个**弱引用对象**储存在集合中  
WeakSet 和 Set 的区别：
* `WeakSet`只能储存对象引用，不能储存其它类型的值，而`Set`都可以。
* `WeakSet`中储存的值都是被弱引用的，即垃圾回收机制不考虑`WeakSet`对该对象的引用，如果没有其他变量或属性值引用这个对象值，那么垃圾回收机制会自动回收该对象占用的内存，不考虑该对象还存在于`WeakSet`之中。  
    所以，`WeakSet`中有多少个成员元素，取决于垃圾回收机制有没有运行，运行前后成员个数可能不一致，而垃圾回收机制何时运行是不可预测的，因此`ES6`规定`WeakSet`不可遍历。


#### 1. 
2. 下面代码保证了`Foo`的实例方法，只能在`Foo`的实例上调用。这里使用WeakSet的好处是，`foos`对实例的引用，不会被记入内存回收机制，所以删除实例的时候，不用考虑`foos`，也不会出现内存泄漏。
    ```js
    const foos = new WeakSet()
    class Foo {
        constructor() {
            foos.add(this)
        }
        method () {
            if (!foos.has(this)) {
                throw new TypeError('Foo.prototype.method 只能在Foo的实例上调用！');
            }
        }
    }

    const a = {}
    a.__proto__ = Foo.prototype
    a.method()  // TypeError: Foo.prototype.method 只能在Foo的实例上调用！
    ```

### 三. 字典（Map）

### 四. WeakMap



### 五. 引用
[Set 和 Map 数据结构](http://es6.ruanyifeng.com/#docs/set-map)

[Set、WeakSet、Map及WeakMap](https://github.com/sisterAn/blog/issues/24)