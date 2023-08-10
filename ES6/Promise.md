## Promise详解

### 一. 回调的问题
1. **回调嵌套**
    * 糟糕的嵌套和缩进，代码会更加杂乱
    * 嵌套书写方式和人的线性思考相违和
    * 流程混乱，难以维护和更新

2. **控制反转**
    当你调用了第三方的 API时，使用回调就可能存在众多不确定因素

    * 回调函数执行多次
    * 回调函数没有执行
    * 回调函数有时同步执行有时异步执行

    对于这些情况，你可能都要在回调函数中做些处理，并且每次执行回调函数的时候都要做些处理，这就带来了很多重复的代码。

3. **回调地狱**
    * 代码难以复用
    * 堆栈信息被断开，难以捕捉错误
    * 借助外层变量

Promise 使得以上绝大部分的问题都得到了解决。

### 二. Promise的反模式
1. Promise嵌套
    ```js
    // bad
    loadSomething().then(something) => {
        loadAnotherthing().then(another) => {
            DoSomethingOnThem(something, another)
        }
    }
    ```
    ```js
    // good
    Promise.all([loadSomething(), loadAnotherthing()])
        .then([something, another] => {
            DoSomethingOnThem(...[something, another])
        })
    ```

2. 断开的Promise链
    ```js
    // bad
    function AnAsyncCall() {
        let promise = doSomethingAsync()
        promise.then(() => {
            somethingComplicated()
        })
        return promise
    }
    ```
    ```js
    // good
    function AnAsyncCall() {
        let promise = doSomethingAsync()
        return promise.then(() => {
            somethingComplicated()
        })
    }
    ```

3. catch
    ```js
    // bad
    somethingAsync().then(val => {
        return somethingElseAsync(val)
    }, err => {
        handleMyError(err)
    })
    ```
    ```js
    // good
    somethingAsync().then(val => {
        return somethingElseAsync(val)
    }).catch(err => {
        handleMyError(err)
    })
    ```
### 三. Promise的局限性
1. 错误被吃掉  
    ```js
    let promise = new Promise(() => {
        throw new Error('error')
    })
    console.log(666)    // 666
    ```
    上面的代码会正常的打印 666，说明 Promise 内部的错误不会影响到 Promise 外部的代码，也就是 “吃掉错误”。  
    而正是因为错误被吃掉，Promise 链中的错误很容易被忽略掉，这也是为什么会一般推荐**在 Promise 链的最后添加一个 catch 函**数，因为对于一个没有错误处理函数的 Promise 链，任何错误都会在链中被传播下去，直到你注册了错误处理函数。

2. 单一值  
    Promise 只能有一个完成值或一个拒绝原因，然而在真实使用的时候，往往需要传递多个值，一般做法都是构造一个对象或数组，然后再传递，then 中获得这个值后，又会进行取值赋值的操作，每次封装和解封都无疑让代码变得笨重。  
    并没有什么好的方法，建议是使用 ES6 的解构赋值：
    ```js
    Promise.all([Promise.resolve(1), Promise.resolve(2)])
    .then(([x, y]) => {
        console.log(x, y)
    })
    ```

3. 无法取消  
    Promise 一旦新建它就会立即执行，无法中途取消。

4. 无法得知pending状态  
    当处于 pending 状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

### 四. Promise的小问题
1. 值穿透
    ```js
    Promise.resolve(1)
        .then(2)
        .then(Promise.resolve(3))
        .then(console.log)
    // 输出结果：1
    ```
    `Promise`的`then()`的参数期望是函数，传入非函数则会发生值穿透。  
    当然，上面的结果也是因为紧跟其后的`then()`都没有`return`任何值，如果稍微修改下，结果就不一样了。
    ```js
    Promise.resolve(1)
        .then(() => 2)
        .then(Promise.resolve(3))
        .then(console.log)
    // 输出结果：2

    Promise.resolve(1)
        .then(() => 2)
        .then(() => Promise.resolve(3))
        .then(console.log)
    // 输出结果：3
    ```

### 五. 内部实现原理
#### 1. 个人总结
1. `_resolve()`和`_reject()`的封装，目的在于控制成功和失败回调队列依次执行
2. `then()`的设计，返回一个新的`Promise`。根据内部`status`状态，`pending`时添加对应任务队列；状态改变后，立即执行对应回调函数
3. `resolve`，`reject`，`all`，`race`等静态方法往往需要手写返回一个新的`Promise`
4. `catch`，`finally`原型方法则是返回一个特殊处理后的`then()`

#### 2. 代码模拟实现
[Promise 模拟实现 =>>](../算法&数据结构&手撕代码/JavaScript.md#promise-模拟实现)

### 六. 应用
#### 1. race()
使用`race()`特性做超时的操作
```js
// 请求某个图片资源
var requestImg = new Promise(function(resolve, reject) {
    var img = new Image();
    img.onload = function() {
        resolve(img);
    };
});
// 延时函数，用于给请求计时
var timeOut = new Promise(function(resolve, reject) {
    setTimeout(function() {
        reject('图片请求超时');
    }, 5000);
});
Promise.race([requestImg, timeOut]).then(function(res) {
    console.log(res);
}, function(err) {
    console.log(err);
});
```

### 七. 引用
[Promise实现原理](https://juejin.im/post/5b83cb5ae51d4538cc3ec354)

[ES6 系列之我们来聊聊 Promise](https://github.com/mqyqingfeng/Blog/issues/98)
