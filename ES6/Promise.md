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

```js
const isFunction = variable => typeof variable === 'function'

const PENDING = 'pending'      // 进行中
const FULFILLED = 'fulfilled'  // 已成功
const REJECTED = 'rejected'    // 已失败

class MyPromise {
    constructor(fn) {
        // MyPromise类接受一个函数作为参数
        if (!isFunction(fn)) {
            throw new Error('MyPromise must accept a function as a parameter')
        }
        // 开始状态 pending
        this._status = PENDING
        // 返回值
        this._value = undefined
        // 成功回调函数队列
        this._fulfilledQueues = []
        // 失败回调函数队列
        this._rejectedQueues = []

        // 执行 fn 函数
        try {
            fn(this._resolve.bind(this), this._reject.bind(this))
        } catch (err) {
            this._reject(err)
        }
    }
    _resolve(val) {
        if (this._status !== PENDING) return
        let run = () => {
            // 依次执行成功队列中的函数，并清空队列
            const runFulfilled = (value) => {
                let cb
                while (cb = this._fulfilledQueues.shift()) {
                    cb(value)
                }
            }

            // 依次执行失败队列中的函数，并清空队列
            const runRejected = (error) => {
                let cb
                while (cb = this._rejectedQueues.shift()) {
                    cb(error)
                }
            }

            // 如果resolve的参数为Promise对象，则必须等待Promise对象状态改变后，
            // 当前Promise对象的状态才会改变，且状态取决于参数Promise对象的状态
            if (val instanceof MyPromise) {
                val.then(value => {
                    this._status = FULFILLED
                    this._value = value
                    runFulfilled(value)
                }, error => {
                    this._status = REJECTED
                    this._value = error
                    runRejected(error)
                })
            } else {
                this._status = FULFILLED
                this._value = val
                runFulfilled(val)
            }
        }
        // 为了支持同步的Promise，这里采用异步调用
        setTimeout(run, 0)
    }
    _reject(err) {
        if (this._status !== PENDING) return
        let run = () => {
            this._status = REJECTED
            this._value = err
            // 依次执行成功队列中的函数，并清空队列
            let cb
            while (cb = this._rejectedQueues.shift()) {
                cb(err)
            }
        }
        // 为了支持同步的Promise，这里采用异步调用
        setTimeout(run, 0)
    }
    then(onFulfilled, onRejected) {
        const { _value, _status } = this
        // 返回一个新的 Promise 对象
        return new MyPromise((resolveNext, rejectNext) => {
            // 封装一个成功时执行的函数
            let fulfilled = value => {
                try {
                    if (!isFunction(onFulfilled)) {
                        // 值穿透发生在此处，相当于then方法执行时并未对_value进行修改
                        resolveNext(value)
                    } else {
                        let res = onFulfilled(value)
                        if (res instanceof MyPromise) {
                            // 如果当前回调函数返回Promise对象，必须等待其状态改变后再执行下一回调
                            res.then(resolveNext, rejectNext)
                        } else {
                            // 否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
                            resolveNext(res)
                        }
                    }
                } catch (err) {
                    // 如果新的函数执行出错，新的Promise对象状态为失败
                    rejectNext(err)
                }
            }

            // 封装一个失败时执行的函数
            let rejected = error => {
                try {
                    if (!isFunction(onRejected)) {
                        // 如果失败回调不是函数，会调用当前函数的reject方法
                        rejectNext(error)
                    } else {
                        let res = onRejected(error)
                        if (res instanceof MyPromise) {
                            // 如果当前回调函数返回Promise对象，必须等待其状态改变后再执行下一回调
                            res.then(resolveNext, rejectNext)
                        } else {
                            // 否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
                            resolveNext(res)
                        }
                    }
                } catch (err) {
                    rejectNext(err)
                }
            }

            switch (_status) {
                case PENDING:
                    this._fulfilledQueues.push(fulfilled)
                    this._rejectedQueues.push(rejected)
                    break
                case FULFILLED:
                    fulfilled(_value)
                    break
                case REJECTED:
                    rejected(_value)
                    break
            }
        })
    }
    catch(onRejected) {
        // 实际相当于返回一个只有失败回调的then方法
        return this.then(undefined, onRejected)
    }
    // 添加静态方法 resolve
    static resolve(value) {
        // 如果参数是Promise实例，直接返回这个实例
        if (value instanceof MyPromise) return value
        return new MyPromise(resolve => resolve(value))
    }
    // 添加静态方法 reject
    static reject(value) {
        return new MyPromise((resolve, reject) => reject(value))
    }
    // 添加静态方法 all
    static all(promises) {
        // 返回一个新的Promise实例
        return new MyPromise((resolve, reject) => {
            // 返回有序的值的集合
            let values = []
            let count = 0
            for (let [i, p] of promises.entries()) {
                // 数组参数如果不是Promise实例，先调用Promise.resolve
                this.resolve(p).then(val => {
                    values[i] = val
                    count++
                    // 所有状态都变成fulfilled时，返回的Proimse状态就变成fulfilled
                    if (count === promises.length) resolve(values)
                }, err => {
                    // 只要有一个状态变为rejected，返回的Promise状态就变成rejected
                    reject(err)
                })
            }
        })
    }
    // 添加静态方法 race
    static race(promises) {
        return new MyPromise((resolve, reject) => {
            for (let p of promises) {
                // 只要有一个实例率先改变状态，返回的Promise状态就跟着改变
                this.resolve(p).then(val => {
                    resolve(val)
                }, err => {
                    reject(err)
                })
            }
        })
    }
    // 添加静态方法 finally
    finally(cb) {
        // 通catch方法一样，实际返回了一个then方法
        return this.then(
            // 之所以再包装一层Promise.resolve，是为了保证返回值不变
            // Promise.resolve(3).finally(() => {}) // 3
            val => MyPromise.resolve(cb()).then(() => val),
            err => MyPromise.resolve(cb()).then(() => { throw err })
        )
    }
}
```

### 六. 引用
[Promise实现原理](https://juejin.im/post/5b83cb5ae51d4538cc3ec354)

[ES6 系列之我们来聊聊 Promise](https://github.com/mqyqingfeng/Blog/issues/98)

TODO: 其他的Promise实现，大致思路是一样的，细节再看下（关于值穿透的处理不太一样）

[* 剖析Promise内部结构 *](https://github.com/xieranmaya/blog/issues/3)

[简单实现Promise](https://imweb.io/topic/5bbc264b6477d81e668cc930)

[Promise原理讲解](https://juejin.im/post/5aa7868b6fb9a028dd4de672)