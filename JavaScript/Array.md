## Array 详解

### 一. 常用方法
![数组常用方法](http://img.vanilla.ink/me/webproject/FE-Summary/JavaScript/Array/01.png)

1. 遍历
    1. reduce()
        可用于替代 filter() + map() 用于同时筛选和处理新的返回数组
        ```js
        [1, 2, 3, 4].reduce((acc, item, index, arr) => {
            return acc + item
        }, 5)
        ```
2. 查找
    2. includes()
        用于代替[].indexOf() > -1 的写法
        ```js
        [1, 2, 3, 4].includes(2)
        // true
        [1, 2, 3, 4].includes(8)
        // false
        ```

### 二. 奇巧淫技
1. **生成类似`[1-100]`这样的数组**  
    ```js
    const add = new Array(100).fill(0).map((item, index) => index + 1)
    ```

2. **数组解构赋值的应用**  
    交换变量
    ```js
    let a = 1,
        b = 2

    // bad
    let temp = a
    a = b
    b = temp

    // good
    [b, a] = [a, b]

    // 这样也可以
    [o.a, o.b] = [o.b, o.a]
    ```
    生成剩余数组
    ```js
    const [a, ...rest] = [...'asdf'] // a: 'a', rest: ['s', 'd', 'f']
    ```

3. **数组浅拷贝**  
    ```js
    const arr = [1, 2, 3]
    const arrClone = [...arr]
    // 对象也可以这样浅拷贝
    const obj = { a: 1 }
    const objClone = { ...obj }
    ```

4. **数组合并**  
    ```js
    // 除了 concat 方法，还可以使用 ... 操作符
    const arr1 = [1, 2]
    const arr2 = [3, 4]
    const arr = [...arr1, ...arr2] // [1, 2, 3, 4]
    ```

5. **数组去重**  
    ```js
    // ser 数据类型的元素不会重复，利用这个特性来去重
    const arr = [1, 1, 2, 2, 3, 4, 5, 5]
    const newArr = [...new Set(arr)] // [1, 2, 3, 4, 5]
    ```

6. **数组取交集**  
    ```js
    const a = [0, 1, 2, 3, 4, 5]
    const b = [3, 4, 5, 6, 7, 8]
    const arr = [...new Set(a)].filter(item => b.includes(item)) // [3, 4, 5]
    ```

7. **数组取差集**  
    ```js
    const a = [0, 1, 2, 3, 4, 5]
    const b = [3, 4, 5, 6, 7, 8]
    const arr = [...new Set([...a, ...b])].filter(item => !a.includes(item) || !b.includes(item)) // [0, 1, 2, 6, 7, 8]
    ```

8. **使用 includes 简化 if 判断**
    ```js
    // bad
    if (a === 1 || a === 2 || a === 3 || a === 4) {
        // todo
    }

    // good
    let arr = [1, 2, 3, 4]
    if (arr.includes(a)) {
        // todo
    }
    ```

9. **使用 some 方法判断是否有满足条件的元素**
    ```js
    let arr = [1, 3, 5, 7]

    // bad
    function isHasNum (n) {
        for (let i = 0; i < arr.length; i++) {
            if (arr[i] === n) {
                return true
            }
        }
        return false
    }

    // good
    let isHasNum = n => arr.some(num => num === n)

    // best
    let isHasNum = (n arr) => arr.some(num => num === n)
    ```

### 三. 数组方法模拟实现
1. 模拟实现一个 forEach  
    ```js
    if (!Array.prototype.forEach) {
        Array.prototype.forEach = function(fn, thisObj) {
            var scope = thisObj || window;
            for (var i = 0, len = this.length; i < len; i++) {
                fn.call(scope, this[i], i, this);
            }
        }
    }
    ```

2. 模拟实现一个 filter  
    ```js
    if (!Array.prototype.filter) {
        Array.prototype.filter = function(fn, thisObj) {
            var scope = thisObj || window,
                res = [];
            for (var i = 0, len = this.length; i < len; i++) {
                if (fn.call(scope, this[i], i, this)) {
                    res.push(this[i]);
                }
            }
            return res;
        }
    }
    ```

### 四. 数组相关算法
1. 乱序  
    [关于数组乱序的深挖——“感觉一直在写毒代码”](https://www.jianshu.com/p/46cfd493964a)
    [前端面试常见问题之『数组乱序』](https://segmentfault.com/a/1190000005875191)

2. 给定两个数组，计算它们的交集
    
    ```js
    // 哈希表，时间复杂度O(m + n) m为arr1长度，n为arr2长度
    let arr1 = [1, 2, 3, 2, 2, '5', '你好啊', 7],
    arr2 = [2, 7, 9, 3, 'hello', '你好啊', 120];

    const interset = (arr1, arr2) => {
        let map = {}
        let res = []
        for (let key of arr1) {
            if (map[key] > 0) {
                map[key]++
            } else {
                map[key] = 1
            }
        }
        for (let key of arr2) {
            if (map[key] > 0) {
                res.push(key)
            }
        }
        return res
    }
    ```
3. 去重

### 五. 参考
1. [JS数组奇巧淫技](https://mp.weixin.qq.com/s/c9jtemK9Do85RG4ub1MORA)

[Array.prototype.reduce 实用指南](https://juejin.im/post/5bab8a9c6fb9a05d0e2e6bf0)
[如何在 JavaScript 中更好地使用数组](https://juejin.im/post/5b8d0a74f265da431d0e7ec0)
[25个你不得不知道的数组reduce高级用法](https://juejin.cn/post/6844904063729926152)