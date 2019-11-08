## JS异步编程解决方案 & 发展历程 & 优缺点

### 一. 为什么需要异步
JS的“单线程”特性，避免执行阻塞。

### 二. 异步解决方案

#### 1. 回调函数callback
如`setTimeout`，`ajax`，`readFile`...
```js
ajax(url, () => {
    // 处理逻辑
})
```
**- 优点：**  
1. 简单，容易理解和实现

**- 缺点：**  
1. 不利于代码阅读和维护
2. 回调地狱
    各部分之间高度耦合，程序结构混乱，流程难以追踪（尤其是多个回调函数嵌套的情况）
    ```js
    ajax(url, () => {
        // 处理逻辑
        ajax(url1, () => {
            // 处理逻辑
            ajax(url2, () => {
                // 处理逻辑
            })
        })    
    })
    ```
3. 每个任务只能指定一个回调函数
4. 不能使用`try/catch`捕获错误，不能直接`return`

#### 2. 事件发布订阅
如Node中的events，Dom事件绑定。

首先，f2向信号中心jQuery订阅`done`信号
```js
jQuery.subscribe('done' ,f2);
```
f1代码如下
```js
function f1() {
    ajax(url, () => {
        // ...
        jQuery.publish('done');
    });
}
```
f1执行完成后，向信号中心jQuery发布done信号，引发f2的执行，f2执行完之后，可以取消订阅
```js
jQuery.unsubscribe('done', f2);
```
**- 优点：**  
1. 相对于回调函数，一定程度上实现了解耦

#### 3. Promise/A+
```js
function request(url) {
    return new Promise((resolve, reject) => {
        ajax(url, (res) => {
            resolve(res)
        }, (err) => {
            reject(err)
        })
    })
}
request(url)
    .then(() => {
        return request(url1)
    })
    .then(() => {
        return request(url2)
    })
    .catch((err) => {
        console.log(err)
    })
```
**- 优点：**  
1. 可以捕捉错误
2. 很好地解决了回调地狱问题

**- 缺点：**  
1. 无法取消`Promise`。一旦新建，它就会立即执行，无法中途取消。
2. 错误需要通过回调函数捕捉。如果不设置回调函数，`Promise`内部抛出的错误不会反应到外部。
3. 当处于`pending`状态时，无法得知目前进展到哪一个阶段（刚开始还是即将完成）。

#### 4. 生成器Generator
```js
function* fetch() {
    yield ajax(url, () => {});
    yield ajax(url1, () => {});
    yield ajax(url2, () => {});
}
let it = fetch();
let result1 = it.next();
let result2 = it.next();
let result3 = it.next();
```
**- 优点：**  
1. 解决了回调地狱问题，异步操作的写法更像是同步操作，代码更加清晰

**- 缺点：**  
1. 不能自动执行异步操作，需要写多个`next()`方法，手动迭代`Generator`函数很麻烦，一般需要配合`co`库去使用

#### 5. Async/Await
1. 简介
    * 可以理解为`Generator`的语法糖，`async`等于`Generator`和`co`模块（内部也使用了`Promise`）的封装，`async`函数返回一个`Promise`对象
    * 与`Promise`一样，是非阻塞的
    * 使得异步代码看起来像同步代码

2. 依次调用
    ```js
    async function fetch() {
        try {
            let res1 = await ajax(url, () => {});   //await后面跟的是一个Promise实例
            let res2 = await ajax(url1, () => {});
            let res3 = await ajax(url2, () => {});
            console.log(res1);
            console.log(res2);
            console.log(res3);
            return res3;
        } carch(error) {
            console.log(error);
        }
    }
    fetch().then((res) => { // async函数返回的也是个Promise
        console.log(res);
    }).catch((err) => {
        console.log(err);
    });
    ```

3. 并发请求
    ```js
    // 如果两个请求毫无关系，可以通过并发请求
    function fetchAll() {
        fetch1();
        fetch2();  //这个函数同步执行
    }
    async function fetch1() {
        let r = await ajax(url1, () => {});
        console.log(r);
    }
    async function fetch2() {
        let r = await ajax(url2, () => {});
        console.log(r);
    }
    fetchAll();
    ```
**- 优点：**  
1. 可以捕获同步和异步错误
2. `async/await`相对于`Promise`优势体现在：
    * 处理then的调用链，能够更清晰准确地写出代码
    * 并且也更优雅地解决回调地狱问题
3. `async/await`对`Generator`函数的改进：
    * 内置执行器。`Generator`函数的执行必须靠执行器，所以才有了`co`函数库，而`async`函数自带执行器。
    * 更广的适用性。`co`函数库约定，`yield`命令后面只能是`Thunk`函数或者`Promise`对象，而`async`函数的`await`命令后，可以跟`Promise`对象和原始类型的值（数值，字符串和布尔值，但这等同于同步操作）
    * 更好的语义。`async`和`await`比起`*`和`yield`，语义更清楚了。`async`表示函数里有异步操作，`await`表示紧跟在后面的表达式需要等待结果

**- 缺点：**  
1. `await`将异步代码改造成了同步代码，如果多段异步代码没有依赖性却使用了`await`会导致性能上的降低。代码没有依赖性的话，可以使用`Promise.all()`的方式

### 三. 总结
1. 进化史：callback -> 发布订阅 -> `Promise` -> `Generator` -> `async/await`
2. `async/await`函数的实现，就是将`Generator`函数和自动执行器，包装在一个函数里
3. `async/await`可以说是异步终极解决方案了

### 四. 引用
[JS 异步编程六种方案](https://juejin.im/post/5c30375851882525ec200027)