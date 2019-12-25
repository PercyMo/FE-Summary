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


#### 1. 实例属性
* `constructor`: 构造函数，任何一个具有`Iterable`接口的对象，都可以作为参数
    ```js
    const arr = [[1, 2], [3, 4]]
    const ws = new WeakSet(arr)
    console.log(ws)     // WeakSet {Array(2), Array(2)}
    ```

#### 2. 实例方法
同 `Set` 类似，拥有`add()`，`has()`，`delete()`，不同的是，没有`clear()`

#### 3. 例举一种使用场景
下面代码保证了`Foo`的实例方法，只能在`Foo`的实例上调用。这里使用WeakSet的好处是，`foos`对实例的引用，不会被记入内存回收机制，所以删除实例的时候，不用考虑`foos`，也不会出现内存泄漏。
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
集合 和 字典 的区别：
* 相同点：集合、字典可以储存不重复的值
* 不同点：集合是以`[value, value]`的形式储存元素，字典是以`[key, value]`的形式储存
**任何具有`Iterator`接口，且每个成员都是双元素数组的数据结构**，都可以当做`Map`构造函数的参数，例如：
```js
const s = new Set([
    ['foo', 1],
    ['bar', 2]
])
const m1 = new Map(set)
m1.get('foo')   // 1

const m2 = new Map([['baz', 3]])
const m3 = new Map(m2)
m3.get('baz')   // 3
```

如果读取一个未知的键，则返回`undefined`
```js
new Map().get('asdasfa')
// undefined
```

**注意：只有对同一个对象的引用，Map结构才将其视为同一个键。这一点要非常小心。**
```js
const map = new Map()
map.set(['a'], 5)
map.get(['a'])      // undefined
```
上面代码的`get`和`set`方法，表面是针对同一个键，但实际上是两个值，内存地址是不一样的，因此`get`方法无法读取该值，返回`undefined`。  
由上可知，`Map`的键是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞（`clash`）的问题，我们扩展别人的库的时候，如果使用对象作为键名，就不用担心自己的属性与原作者的属性同名。  
如果`Map`的键是一个简单类型的值（数字，字符串，布尔值），则只要两个值严格相等，`Map`将其视为一个键，比如`0`和`-0`就是一个键，布尔值`true`和字符串`true`则是两个不同的键。另外`undefined`和`null`也是两个不同的键。虽然`NaN`不严格相等于自身，但`Map`将其视为同一个键。
```js
const map = new Map()

map.set(-0, 123)
map.get(+0)      // 123

map.set(true, 1)
map.set('true', 2)
map.get(true)   // 1

map.set(undefined, 3)
map.set(null, 4)
map.get(undefined)  // 3

map.set(NaN, 123)
map.get(NaN)    // 123
```

#### 1. 属性与方法
1. 属性
    * constructor: 构造函数
    * size: 字典中包含的元素个数

2. 操作方法
    `set()`，`get()`，`has()`，`delete()`，`clear()`

3. 遍历方法
    `keys()`，`values()`，`entries()`，`forEach()`
    ```js
    const map = new Map([
        ['name', 'An'],
        ['des', 'JS']
    ])
    console.log(map.entries())  // MapIterator {"name" => "An", "des" => "JS"}
    console.log(map.keys())     // MapIterator {"name", "des"}
    ```
    `Map`结构的默认遍历器接口（`Symbol.iterator`属性），就是`entries`方法
    ```js
    Map.prototype[Symbol.iterator] === Map.prototype.entries
    // true
    ```

#### 2. 与其他数据结构的相互转换
1. Map 转 Array
    ```js
    const map = new Map([[1, 1], [2, 2], [3, 3]])
    console.log([...map])   // [[1, 1], [2, 2], [3, 3]]
    ```
2. Array 转 Map
    ```js
    const map = new Map([[1, 1], [2, 2], [3, 3]])
    console.log(map)        // Map(3) {1 => 1, 2 => 2, 3 => 3}
    ```
3. Map 转 Object
    ```js
    function mapToObj (map) {
        const obj = Object.create(null)
        for (let [key, value] of map) {
            obj[key] = value
        }
        return obj
    }
    const map = new Map().set('name', 'An').set('des', 'JS')
    mapToObj(map)   // {name: "An", des: "JS"}
    ```
4. Object 转 Map
    ```js
    function objToMap(obj) {
        const map = new Map()
        for (let key of Object.keys(obj)) {
            map.set(key, obj[key])
        }
        return map
    }
    objToMap({name: "An", des: "JS"})   // Map(2) {"name" => "An", "des" => "JS"}
    ```
5. Map 转 JSON
    ```js
    function mapToJson(map) {
        return JSON.stringify([...map])
    }
    const map = new Map().set('name', 'An').set('des', 'JS')
    mapToJson(map)      // [["name","An"],["des","JS"]]
    ```
6. JSON 转 Map
    ```js
    function JsonToMap(jsonStr) {
        return objToMap(JSON.parse(jsonStr))
    }
    JsonToMap('{"name":"An","des":"JS"}')   // Map(2) {"name" => "An", "des" => "JS"}
    ```

### 四. WeakMap
`WeakMap` 是一组键值对的集合，其中的**键是弱引用对象，而值可以是任意**的。  
**注意：`WeakMap` 弱引用的只是键名，而不是键值。键值依然是正常引用。**  
`WeakMap` 中，键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。`WeakMap`里面的键名对象和键值对会自动消失。所以，`WeakMap` 的 `key` 是不可枚举的。

#### 1. 属性和方法
1. 属性
constructor: 构造函数

2. 方法
`has()`，`set`，`get()`，`delete()`

#### 2. 例举两种使用场景
1. 使用`WeakMap`的键值储存`DOM`节点，可以避免节点删除时，内存泄漏的风险
    ```js
    const el = document.getElementById('logo')
    const wm = new WeakMap()

    wm.set(el, {count: 0})
    el.addEventListener('click', function() {
        const data = wm.get(el)
        data.count++
    }, false)
    ```
2. 使用`WeakMap`部署私有属性
    ```js
    const _counter = new WeakMap()
    const _action = new WeakMap()
    class Countdown {
        constructor (counter, action) {
            _counter.set(this, counter)
            _action.set(this, action)
        }
        dec () {
            let counter = _counter.get(this)
            if (counter < 1) return
            counter--
            _counter.set(this, counter)
            if (counter === 0) {
                _action.get(this)()
            }
        }
    }
    const c = new Countdown(2, () => console.log('DONE'))

    c.dec()
    c.dec()
    // DONE
    ```

### 五. 验证WeakSet和WeakMap
可以在Node中，或者浏览器控制台的Memory工具下通过内存快照观察内存释放的情况。
```sh
# --expose-gc参数表示允许手动执行垃圾回收机制
$ node --expose-gc
```
```sh
// 手动执行一次垃圾回收，保证获取的内存使用状态准确
> global.gc();
undefined

// 查看内存占用的初始状态，heapUsed 为 4M 左右
> process.memoryUsage();
{ rss: 21106688,
  heapTotal: 7376896,
  heapUsed: 4153936,
  external: 9059 }

> let wm = new WeakMap();
undefined

// 新建一个变量 key，指向一个 5*1024*1024 的数组
> let key = new Array(5 * 1024 * 1024);
undefined

// 设置 WeakMap 实例的键名，也指向 key 数组
// 这时，key 数组实际被引用了两次，
// 变量 key 引用一次，WeakMap 的键名引用了第二次
// 但是，WeakMap 是弱引用，对于引擎来说，引用计数还是1
> wm.set(key, 1);
WeakMap {}

> global.gc();
undefined

// 这时内存占用 heapUsed 增加到 45M 了
> process.memoryUsage();
{ rss: 67538944,
  heapTotal: 7376896,
  heapUsed: 45782816,
  external: 8945 }

// 清除变量 key 对数组的引用，
// 但没有手动清除 WeakMap 实例的键名对数组的引用
> key = null;
null

// 再次执行垃圾回收
> global.gc();
undefined

// 内存占用 heapUsed 变回 4M 左右，
// 可以看到 WeakMap 的键名引用没有阻止 gc 对内存的回收
> process.memoryUsage();
{ rss: 20639744,
  heapTotal: 8425472,
  heapUsed: 3979792,
  external: 8956 }
```

### 六. 总结
* Set
    * 成员唯一，无序且不重复
    * [value, value]，键值与键名是一致的，（或者说只有键值没有键名）
    * 可以遍历，方法有`add`、`delete`、`has`
* WeakSet
    * 成员都是对象
    * 成员都是弱引用，引用会被垃圾回收机制忽略，可以用来保存DOM节点，避免内存泄漏
    * 不能遍历，有方法`add`、`delete`、`has`
* Map
    * 本质是键值对的集合，类似对象
    * 可以遍历
* WeakMap
    * 只接受对象作为键名，不接受其他类型作为键名
    * 键名是弱引用，键值可以是任意的，键名对对象的引用会被垃圾回收机制忽略，键值不会
    * 不能遍历，方法有`set`，`get`，`delete`，`has`

### 七. 引用
[Set 和 Map 数据结构](http://es6.ruanyifeng.com/#docs/set-map)

[Set、WeakSet、Map及WeakMap](https://github.com/sisterAn/blog/issues/24)