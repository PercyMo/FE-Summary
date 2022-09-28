## 浅拷贝 & 深拷贝

### 一. 浅拷贝和深拷贝
1. **浅拷贝**

![浅拷贝](http://img.vanilla.ink/me/webproject/FE-Summary/Algorithm/DeepClone/01.png?x-oss-process=image/resize,w_600)

创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址，所以如果其中一个对象改变了这个地址，就会影响到另一个对象。

2. **深拷贝**

![深拷贝](http://img.vanilla.ink/me/webproject/FE-Summary/Algorithm/DeepClone/02.png?x-oss-process=image/resize,w_600)

讲一个对象从内存中完整的拷贝一份出来，从堆内存中开辟一个新的区域存放新对象，且修改新对象不会影响原对象。

### 二. 浅拷贝的实现方式
1. `Object.assign()`
    ```js
    var obj = { a: {a: 'kobe', b: 39} }
    var initalObj = Object.assign({}, obj)
    initalObj.a.a = 'wade'
    console.log(obj.a.a)   // wade
    ```
    **注意：当object只有一层的时候，是深拷贝**

2. `Array.prototype.concat()`
    ```js
    var arr = [1, 3, {
        userName: 'percy'
    }];
    var arr2 = arr.concat()
    arr2[2].userName = 'wade'
    console.log(arr[2].userName)    // wade
    ```

3. `Array.prototype.slice()`
    ```js
    var arr = [1, 3, {
        userName: 'percy'
    }];
    var arr2 = arr.slice()
    arr2[2].userName = 'wade'
    console.log(arr[2].userName)    // wade
    ```

4. 展开语法`Spread`
    ```js
    var m = {
        a: 1,
        b: {
            name: 'mao'
        }
    }
    var n = {...m}
    m.b.name = 'percy'
    console.log(n.b.name)    // 'percy'
    ```

### 三. 深拷贝的实现方式
1. `JSON.parse(JSON.stringify())`
    ```js
    var arr = [1, 3, {
        userName: 'percy'
    }, function() {}];
    var arr2 = JSON.parse(JSON.stringify(arr))
    arr2[2].userName = 'wade'
    console.log(arr[2].userName)    // percy
    ```
    **这种方法虽然可以实现数组和对象的深拷贝，但不能处理函数**
    ```js
    var arr = [1, 3, {
        userName: 'percy'
    }, function() {}];
    var arr2 = JSON.parse(JSON.stringify(arr))
    arr2    // [1, 3, {…}, null]
    ```

2. 深度优先，递归遍历  
    [深浅拷贝 =>>](../算法&数据结构&手撕代码/JavaScript.md#深浅拷贝)

3. 广度优先  
    使用队列，`while`进行遍历
    ```js
    function getInit(target) {
        const type = Object.prototype.toString.call(target)
        if (type === '[object Object]') {
            return {}
        } else if (type === '[object Array]') {
            return []
        }
        return target
    }

    function cloneBFS(target) {
        const queue = []
        const map = new Map()   // 记录出现过的对象

        const cloneTarget = getInit(target)
        // 记录下引用类型，并添加到任务队列
        if (cloneTarget !== target) {
            queue.push([target, cloneTarget])
            map.set(target, cloneTarget)
        }

        while (queue.length) {
            const [tar, ctar] = queue.shift()
            for (let key in tar) {
                // 处理循环引用
                if (map.get(tar[key])) {
                    ctar[key] = map.get(tar[key])
                    continue
                }
                
                ctar[key] = getInit(tar[key])
                // 处理引用类型
                if (ctar[key] !== tar[key]) {
                    queue.push([tar[key], ctar[key]])
                    map.set(tar[key], ctar[key])
                }
            }
        }

        return cloneTarget
    }
    ```

### 四. lodash中deepclone源码分析
// TODO: 有点费时间，后续再补全
* 位掩码
* 定制 clone 函数
* 非对象
* 数组 & 正则
* 对象 & 函数
* 循环引用
* Map & Set
* Symbol & 原型链


### 五. 总结
通过深拷贝可以考察到什么
1. 基本实现
    * 递归能力
    * 深度优先和广度优先两种遍历思想
2. 循环引用
    * 考虑问题的全面性
    * 理解weakmap的真正意义
3. 多种类型
    * 考虑问题的严谨性
    * 创建各种引用类型的方法，JS API的熟练程度
    * 准确判断数据类型，对数据类型的理解程度
4. 通用遍历
    * 写代码可以考虑性能优化
    * 了解几种遍历的效率
    * 代码抽象能力
5. 拷贝函数
    * 箭头函数和普通函数的区别（箭头函数没有`prototype`）

### 六. 引用

[如何写出一个惊艳面试官的深拷贝?](https://juejin.im/post/5d6aa4f96fb9a06b112ad5b1)

[Lodash是如何实现深拷贝的](https://github.com/yygmind/blog/issues/31)
