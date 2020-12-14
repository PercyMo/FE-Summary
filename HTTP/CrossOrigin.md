## 前端跨域问题详解

### 1. 什么是跨域
违背了“同源策略”。简单理解就是，要请求的资源与该资源本身所在的服务器，`协议、域名、端口`任意一种不同时。

注意：
1. 如果是协议和端口造成的跨域，“前台”是无能为力的
2. 在跨域问题上，域仅仅是通过“URL首部”来识别，而不会尝试去判断相同ip地址对应着两个域或者两个域是否在同一个ip上

“URL首部”指`window.location.protocol + window.location.host`

### 2. 跨域常见解决方案
#### 1) JSONP
1. 原理
`<script>、<link>、<img>`不受同源策略限制。

2. 简易实现
    ```js
    // 客户端
    let script = document.createElement('script')
    script.src = 'http://test.com/getData?callback=handleJsonp'
    document.body.appendChild(script)

    // 接收跨域数据的回调函数
    function handleJsonp(data) {
        console.log(data)
    }
    ```
    ```js
    // 后端处理 (koa2 + koa-router)
    router.get('/getData', async (ctx, next) => {
        // 获取jsonp的 callback 约定函数名
        let callbackName = ctx.query.callback || 'callback'
        let resData = {
            success: true,
            data: {
                text: 'jsonp跨域获取数据！'
            }
        }
        // jsonp的 script 字符串
        let jsonpStr = `;${callbackName}(${JSON.stringify(resData)})`
        ctx.type = 'text/script'
        ctx.body = jsonpStr
    })
    ```

3. 缺点
    * 只能实现GET请求
    * 不安全，可能会遭受XSS攻击

4. 补充一点`form`表单跨域  
    `form`表单跨域和`jsonp`跨域其实都是利用了浏览器的历史兼容性，`form`表单会刷新页面不会把结果返回给js，浏览器认为这是安全的，所以没做跨域限制。

#### 2) 跨域资源共享(CORS)
跨域资源共享标准新增了一组HTTP首部字段，允许服务器声明哪些源站通过通过浏览器有权访问哪些资源。
1. 简单请求  
    不会触发CORS预检的请求。
    1. 使用 `GET、POST、HEAD` 之一请求
    
    2. 不得人为设置该集合之外的其他首部字段 ( TODO: 没明白什么意思)
        * Accept
        * Accept-Language
        * Content-Language
        * Content-Type （需要注意额外的限制）
        * DPR
        * Downlink
        * Save-Data
        * Viewport-Width
        * Width

    3. `Content-Type` 的值仅限于下列三者
        * text/plain
        * multipart/form-data
        * application/x-www-form-urlencoded

    4. 请求中的任意`XMLHttpRequestUpload`对象均没有注册事件监听器。(TODO: 待验证)
    
    5. 请求中没有使用`ReadableStream`对象(TODO: 待验证)
2. 预检请求  
    即“非简单请求”的请求，要求必须首先使用`OPTIONS`方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求。
    “预检请求”的使用，可以避免跨域请求对服务器的用户数据产生未预知的影响。
3. 请求附带身份凭证 -> cookie  
    ```js
    // 前端设置是否带cookie
    xhr.withCredentials = true
    ```
    如果服务器端未设置`Control-Allow-Credentials: true`，则响应内容不会返回给请求的发起者
4. 后端处理  
    ```js
    // koa2
    app.use(async (ctx, next) => {
        // 允许跨域访问的域名：若有端口需写全（协议+域名+端口）
        // * 表示不做域名限制
        ctx.set('Access-Control-Allow-Origin', 'https://www.baidu.com')
        // 请求允许使用的HTTP方法
        ctx.set('Access-Control-Allow-Methods', 'OPTIONS, GET, PUT, POST, DELETE')
        // 允许前端带认证cookie：启用此项后，上面的域名不能为'*'，必须指定具体的域名，否则浏览器会提示
        ctx.set('Access-Control-Allow-Credentials', true)
        // 表明该响应的有效时间为 86400 秒，也就是 24 小时。在有效时间内，浏览器无须为同一请求再次发起预检请求
        // ** 这也是一个优化项，可以节省不少请求次数
        ctx.set('Access-Control-Max-Age', 86400)
        
        if (ctx.method === 'OPTIONS') {
            ctx.response.status = 200 // OPTIONS请求不做任何处理
            ctx.body = ''
        } else {
            await next()
        }
    })
    ```
5. 请求过程  
    js进行一次跨域ajax请求，浏览器判断是否为“简单请求”，简单请求就直接发起正式请求，复杂请求就先发送一次预检请求，预检成功后再发送正式请求。

    ![ajax请求过程](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/02.png?x-oss-process=image/resize,w_500)  

#### 3) WebSocket
`WebSocket`是HTML5一种新的协议，支持浏览器与服务器全双工通讯，同时允许跨域通讯。

#### 4) Node中间件代理（两次跨域）
// TODO: 待验证
实现原理：同源策略是浏览器需要遵循的标准，而如果是服务器向服务器请求就无需遵循同源策略。代理服务器需要做一下几个步骤：
* 接受客户端请求
* 将请求转发给服务器
* 拿到服务器响应数据
* 将响应转发给客户端

    ![Node中间件代理（两次跨域）](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/03.jpg?x-oss-process=image/resize,w_500)  

```js
// node代码
var proxy = require('http-proxy-middleware')

app.use('/', proxy({
    // 代理跨域目标接口
    target: 'https://www.baidu.com',
    changeOrigin: true,

    // 修改响应头信息，实现跨域并允许带cookie
    onProxyRes: function(proxyRes, req, res) {
        res.header('Access-Control-Allow-Origin', 'https://vanilla.ink')
        res.header('Access-Control-Allow-Credentials', true)
    }
}))
```
#### 5) Nginx反向代理
实现原理类似于Node中间件代理，搭建一个中转nginx服务器，用于转发请求。

实现思路：通过nginx配置一个代理服务器（域名与domain1相同，端口不同）做跳板机，反向代理domain2接口，并可以顺便修改cookie中domain信息，方便当前域cookie写入，实现跨域登录。

```nginx
// proxy服务器
server {
    listen       81;
    server_name  www.domain1.com;
    location / {
        proxy_pass   http://www.domain2.com:8080;  #反向代理
        proxy_cookie_domain www.domain2.com www.domain1.com; #修改cookie里域名
        index  index.html index.htm;

        # 当用webpack-dev-server等中间件代理接口访问nignx时，此时无浏览器参与，故没有同源限制，下面的跨域配置可不启用
        add_header Access-Control-Allow-Origin http://www.domain1.com;  #当前端只跨域不带cookie时，可为*
        add_header Access-Control-Allow-Credentials true;
    }
}
```

#### 6) 跨域通信
这几种方式应该属于跨域通讯，和页面间通讯方式有重复内容。

1. `postMessage()`  
    HTML5的新API，允许来自不同源的脚本采用异步方式进行有限的通信，可以实现跨文档，多窗口，跨域消息传递。

2. `window.name + iframe`  
   `window.name` 属性的独特之处：name值在不同的页面（甚至不同域名）加载后依然存在，并且可以支持非常长的name值(2MB)

3. `location.hash + iframe`  
    实现原理：a.html欲与c.html跨域相互通信，通过中间页b.html来实现，三个页面，不同域之间通过iframe的`location.hash`传值，相同域之间直接js访问来通信。

4. `document.domain + iframe`  
    该方式只能用于二级域名相同的情况下，比如`a.test.com`和`b.test.com`适用于该方式。只需要给页面添加 `document.domain = 'test.com'`表示二级域名都相同就可以实现跨域。

    实现原理：两个页面都通过js强制设置`document.domain`为基础主域，就实现了同域。

### 3. CSRF 跨站请求伪造
TODO: [CORS与CSRF](https://my.oschina.net/hosee/blog/903665)

### 4. 引用
[前端跨域问题及其解决方案](https://yq.aliyun.com/articles/610080)

[HTTP访问控制（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)