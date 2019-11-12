
TODO: 具体内容待填充...
### 方法
1. 遍历
    1. forEach()
    2. map()
    3. filter()
    4. reduce()
        可用于替代 filter() + map() 用于同时筛选和处理新的返回数组
        ```js
        [1, 2, 3, 4].reduce((acc, item, index, arr) => {
            return acc + item
        }, 5)
        ```
2. 查找
    1. indexOf()
        ```js
        [1, 2, 3, 4].indexOf(2)
        // 返回实际索引 1
        [1, 2, 3, 4].indexOf(8)
        // 未找到item -1
        ```
    2. includes()
        用于代替[].indexOf() > -1 的写法
        ```js
        [1, 2, 3, 4].includes(2)
        // true
        [1, 2, 3, 4].includes(8)
        // false
        ```
    3. find()
        ```js
        [{a: 1}, {b: 2}].find((item) => {
            return item.b === 2
        })
        // 返回满足条件的第一个项目
        ```
    4. some()
        ```js
        [1, 2, 3, 4].some((item) => {
            return item === 3
        })
        // true
        ```
3. 增删
    1. push()       // 末尾添加
    2. pop()        // 末尾删除
    3. unshift()    // 前置位添加
    4. shift()      // 前置位删除
4. 连接
    1. concat()     // 连接
    2. join()
    ```js
    [1, 2, 3, 4].join(':')
    // '1:2:3:4'
    ```

### 数组方法模拟实现
1. 模拟实现一个forEach
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
2. 模拟实现一个filter
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

### 数组相关算法
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

### 参考
1. [Array.prototype.reduce 实用指南](https://juejin.im/post/5bab8a9c6fb9a05d0e2e6bf0)
2. [如何在 JavaScript 中更好地使用数组](https://juejin.im/post/5b8d0a74f265da431d0e7ec0)