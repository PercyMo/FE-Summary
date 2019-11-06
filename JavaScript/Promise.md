## Promise详解

### 一. Promise标准
TODO: 具体内容待填充...
三种状态  
pending  
fulfilled  
rejected  

一般来说，不要在then方法里面定义reject状态的回调函数（即then的第二个参数），总是使用catch方法

Promise.prototype.then()  
Promise.prototype.catch()  
Promise.prototype.finally()  

### 二. 内部实现原理
#### 1. 个人总结
1. `_resolve()`和`_reject()`的封装，目的在于控制成功和失败回调队列依次执行
2. `then()`的设计，返回一个新的`Promise`。根据内部`status`状态，`pending`时添加对应任务队列；状态改变后，立即执行对应回调函数
3. `resolve`，`reject`，`all`，`race`等静态方法往往需要手写返回一个新的`Promise`
4. `catch`，`finally`原型方法则是返回一个特殊处理后的`then()`

#### 2. 代码模拟实现
  ```js
  const isFunction = variable => typeof variable === 'function'

  const PENDING  = 'pending'      // 进行中
  const FULFILLED  = 'fulfilled'  // 已成功
  const REJECTED  = 'rejected'    // 已失败

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

### 三. 总结

### 四. 引用
[Promise实现原理](https://juejin.im/post/5b83cb5ae51d4538cc3ec354)

promise标准待整理，参考

[ES6 系列之我们来聊聊 Promise](https://github.com/mqyqingfeng/Blog/issues/98)

[剖析Promise内部结构](https://github.com/xieranmaya/blog/issues/3)

[简单实现Promise](https://imweb.io/topic/5bbc264b6477d81e668cc930)

[Promise原理讲解](https://juejin.im/post/5aa7868b6fb9a028dd4de672)