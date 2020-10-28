## koa框架原理和实现
由Express框架原班人马打造。它几乎所有功能都通过插件实现，所以更加轻量。

### 一. 源码结构
1. application.js  
koa的入口文件，向外导出创建class实例的构造函数，它继承了events。还暴露了一些常用的api，比如toJSON、listen、use等。

2. context.js  
koa的上下文ctx，就是一个简单的对象暴露。重点在于属性的代理。

3. request.js、response.js  
对原生res、req的一些操作

### 二. 四大模块实现
* 封装node http server、创建koa类构造函数
* 构造request、response、context对象
* 中间件机制和剥洋葱模型的实现
* 错误捕获和处理

#### 1. 封装node http server和创建koa类构造函数
基于node原生http模块进行封装，实现服务器应用和端口监听。
```js
const http = require('http')
class Application {
    constructor() {
        this.callbackFunc
    }
    use(fn) {
        this.callbakcFunc = fn
    }
    listen(...args) {
        const server = http.createServer(this.callback())
        server.listen(...args)
    }
    callback() {
        return (req, res) => {
            this.callbackFunc(req, res)
        }
    }
}
module.exports = Application
```

#### 2. 构造request、response、context对象
context就是写koa代码时的ctx相当于全局koa实例上下文this，它连接了request、response两个功能模块，并且暴露给koa实例和中间件等回调函数的参数中，起到承上启下的作用。
```js
// request.js
const url = require('url)

module.exports = {
    get query() {
        return url.parse(this.req.query, true).query
    }
}
```
```js
// response.js
module.exports = {
    get body() {
        // body的读写是对 this._body 进行读写和操作，并没有使用原生的this.res.end
        // 因为我们在编写koa代码时，会对body进行多次的读取和修改
        return this._body
    }
    set body(data) {
        this._body = data
    }
    get status() {
        return this.res.statusCode
    }
    set status(code) {
        if (typeof code !== 'number') {
            throw new Error('statusCode must be a number!')
        } else {
            this.res.statusCode = code
        }
    }
}
```

context.js使用`__defineGetter__`和`__defineSetter__`来代替`setter/getter`每一个属性的读取设置，对常用的request和response方法进行挂载和代理。  
通过`context.query`直接代理了`context.request.query`，`context.body`和`context.status`代理了`context.response.body`和`context.response.status`。
```js
// context.js
const http = require('http')
const context = require('./context')
const request = require('./request')
const response = require('./respones')

constructor() {
    this.context = Object.create(context)
    this.request = Object.create(request)
    this.response = Object.create(response)
}

createContext(req, res) {
    const ctx = Object.create(this.context)
    ctx.request = Object.create(this.request)
    ctx.response = Object.create(this.response)
    ctx.req = ctx.request.req = req
    ctx.res = ctx.response.res = res
    return ctx
}
```

> **ctx的扩展**  
> 我们知道`this.context`是中间件中上下文对象ctx的原型。因此在实际开发中，可以将一些常用的方法挂载到`this.context`上面，这样，在中间件ctx中，可以方便使用这些方法，这个概念叫做ctx的扩展。
> 一个例子是阿里的egg.js，已经把这个扩展机制作为一部分，融入到了框架开发中。
```js
const simpleKoa = require('./application)
const app = new simpleKoa()

// 对ctx进行扩展
app.context.echoData = function(errno = 0, data = null, errmsg = '') {
    this.res.setHeader('Content-Type', 'application/json;charset=utf-8')
    this.body = {
        errno: errno,
        data: data,
        errmsg: errmsg
    }
}

app.use(async ctx => {
    let data = {
        name: 'tom',
        age: 16,
        sex: 'male'
    }
    ctx.echoData(0, data, 'success')
})

app.listen(3000, () => {
    console.log('listenning on 3000')
})
```

#### 3. 中间件机制和剥洋葱模型的实现
koa的中间件机制是一个剥洋葱式的模型，多个中间件通过use放进一个数组队列，然后从外层开始执行，遇到next后进入队列中的下一个中间件，所有中间件执行完后开始回帧，执行队列中之前中间件中未执行的代码部分。  
剥洋葱模型在koa1使用的是`generator + co.js`去实现，koa2使用了`async/await + Promise`去实现。  
```js
compose() {
    return async ctx => {
        // createNext函数的作用就是将上一个中间件的next当参数传给下一个中间件，并将上下文ctx绑定当前中间件
        // 当中间件执行完，调用next()的时候，其实就是去执行下一个中间件
        const createNext = (middleware, oldNext) => {
            return async () => {
                await middleware(ctx, oldNext)
            }
        }
        let next = async () => {
            return Promise.resolve()
        }
        const len = this.middlewares.length
        // 这是一个链式反向递归模型的实现
        // 当循环到第一个中间件的时候，只需要执行一次next()，就能链式地递归调用所有中间件
        for (let i = len - 1; i >= 0; i--) {
            const currentMiddleware = this.middlewares[i]
            next = createNext(currentMiddleware, next)
        }
        await next()
    }
}
callback() {
    return (req, res) => {
        const ctx = this.createContext(req, res)
        const respond = () => this.responseBody(ctx)
        const onerror = err => this.onerror(err, ctx)
        const fn = this.compose()
        return fn(ctx).then(respond).catch(onerror)
    }
}
```

**逐步理解洋葱模型设计思路及过程**  
假设已有3个async函数
```js
async function m1(next) {
    console.log('m1')
    await next()
}

async function m2(next) {
    console.log('m2')
    await next()
}

async function m3() {
    console.log('m3')
}
```
希望能构造出一个函数，实现的效果是让三个函数依次执行。首先考虑m2执行完毕后，`await next()`去执行m3函数，显然需要构造一个next函数，作用是调用m3，然后作为参数传给m2，以此类推。
```js
let next1 = async function() {
    await m3()
}
let next2 = async function() {
    await m2(next1)
}
m1(next2)
// 输出: m1,m2,m3
```

假如async函数增加至n个，产生next的过程可以抽象为一个函数：
```js
function createNext(middleware, oldNext) {
    return async function () {
        await middleware(oldNext)
    }
}

const next1 = createNext(m3, null)
const next2 = createNext(m2, next1)
const next3 = createNext(m1, next2)

next3()
```

进一步精简：
```js
const middlewares = [m1, m2, m3]
const len = middlewares.length

// 最后一个中间件的next设置为一个立即resolve的promise函数
let next = async function() {
    return Promise.resolve()
}
for (let i = len - 1; i >= 0; i--) {
    next = createNext(middlewares[i], next)
}
next()
```

#### 4. 错误捕获和错误处理
1. 每一个中间件代码都是由async包裹着的，而且中间件的执行函数compose返回的也是一个async函数。因此只需使用promise的catch方法，就可以把所有中间件的异常全部捕获到。  

2. koa构造函数继承events模块，通过emit函数触发，通过on监听函数就能订阅和监听框架层面上的错误。  

```js
const EventEmitter = require('event')

class Application extends EventEmitter {
    // ...

    callback() {
        const ctx = this.createContext(req, res)
        const respond = () => this.responseBody(ctx)
        const onerror = err => this.onerror(err, ctx)
        const fn = this.compose()
        // 在这里catch异常，调用onerror方法处理异常
        return fn(ctx).then(respond).catch(onerror)
    }

    onerror(err, ctx) {
        if (err.code === 'ENOENT') {
            ctx.status = 404
        } else {
            ctx.status = 500
        }
        const msg = err.message || 'Internal error'
        ctx.res.end(msg)
        // 触发error事件
        this.emit('error', err)
    }
}
```

### 三. 效果
```js
const koa = require('./koa/application.js')
const app = new koa()

app.on('error', err => {
    console.log('error happends: ', err.stack)
})

app.use(async (ctx, next) => {
    console.log('1')
    await next()
    console.log('5')
})

app.use(async (ctx, next) => {
    console.log('2')
    await next()
    console.log('4')
})

app.use(async (ctx, next) => {
    console.log('3')
    ctx.body = 'hello world!'
})

app.listen(8081, () => {
    console.log('监听：', 8081)
})
```

启动服务后，成功监听8081端口，访问页面后，页面输出`hello world`，终端顺序打印`1 2 3 4 5`。  
中间件抛错时，可以成功监听，做相应处理，返回500状态码。  

### 四. 完整代码实现

### 五. 引用

[KOA2框架原理解析和实现](https://juejin.im/post/6844903709592256525#heading-9)

[从头实现一个koa框架](https://zhuanlan.zhihu.com/p/35040744)