## 集合和字典
集合和字典用于存储不重复的值。

### 一. 集合
```js
class MySet extends Set {
  constructor (...agrs) {
    super(...agrs)
  }
  // 交集
  intersection(otherSet) {
    // return new MySet([...this].filter(value => otherSet.has(value)))
    const newSet = new MySet()
    for (let value of this) {
      if (otherSet.has(value)) {
        newSet.add(value)
      }
    }
    return newSet
  }
  // 差集
  difference(otherSet) {
    // return new MySet([...this].filter(value => !otherSet.has(value)))
    const newSet = new MySet()
    for (let value of this) {
      if (!otherSet.has(value)) {
        newSet.add(value)
      }
    }
    return newSet
  }
  // 并集
  union(otherSet) {
    return new MySet([...this, ...otherSet])
  }
  // 子集
  isSubOf(otherSet) {
    if (this.size > otherSet.size) {
      return false
    } else {
      // return [...this].every(value => otherSet.has(value))
      for (let value of this) {
        if (!otherSet.has(value)) {
          return false
        }
      }
      return true
    }
  }
}

const a = new MySet([1, 2, 3, 4])
const b = new MySet([3, 4, 5, 6])
const c = new MySet([1,2])
a.intersection(b) // Set(2) {3, 4}
a.difference(b) // Set(2) {1, 2}
a.union(b) // Set(6) {1, 2, 3, 4, 5, 6}
c.isSubOf(a) // true
c.isSubOf(b) // false
```
上面注释的三处代码可以使用一行代码实现其函数的功能，但时间复杂度为 O(n²)，相比之下，其下面的代码时间复杂度为 O(n)。因为 `[...this]` 先遍历集合中的所有元素转为数组，然后再做后续操作。

### 二. 字典
常说的“键值对”。在 ES6 中的  实现为 `Map`。  
1. `Map` 是有序的
2. `Map` 的键可以是任意类型的值
```js
const o = { 2: 2, 1: 1}
Object.keys(o) // ['1', '2']  键只能是无序的字符串
const map = new Map()
map.set(o, 1)
map.set(2, 2)
for (let key of map) {
  console.log(key) // [{…}, 1] [2, 2]  键可以是任意类型，且根据添加的顺序遍历
}
```
如果为集合添加了一个没有引用的对象的话，就再也取不到这个对象了：
```js
const m = new MySet()
m.add({})
m.has({}) // false

const n = new Map()
n.set({}, 1)
n.get({}) // undefined
```
