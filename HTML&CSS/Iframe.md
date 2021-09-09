## iframe
### 一. iframe 常用属性
* frameborder: 是否显示边框：1 - 显示，0 - 不显示
* height: 作为普通元素的高度，建议在css设置
* width: 作为普通元素的宽度，建议在css设置
* name: 框架的名字，`window.frames[name]` 是专用的属性
* scrolling: 框架是否滚动：yes，no，auto
* src: 内框架地址
* srcdoc: 用来替代原 HTML body 中的内容。（想到一种场景，接口返回大段HTML，直接渲染进去，不用额外新建一个页面）
* sandbox: 对 iframe 进行一些限制，IE10+支持

同域时，父页面可以对子页面进行改写，反之亦然。  
不同域时，父页面没有权限改动子页面，但可以实现页面跳转。

### 二. 获取 iframe 内容
#### 1. 获取页面内嵌 iframe 内容
```js
// 通过name获取，推荐这种方式，代码比较简洁
const iframe = window.frames.iframe1
const iwindow = iframe.window
const idoc = iwindow.document

// 通过id获取
const iframe = document.getElementById('iframe1')
const iwindow = iframe.contentWindow
const idoc = iwindow.document
```

#### 2. 在 iframe 中获取父级内容
```js
window.parent // 获取上一级window对象，如果还是iframe则是该iframe的window对象
window.top // 获取最顶级容器的window对象，就是你打开页面的文档
window.self // 返回自身window的引用，window === window.self（不知道设计这个API有什么卵用）
```

### 三. iframe 长轮询
很古老的一种做法
```js
const iframeCon = document.querySelector('#container')
let text = null // 传递的信息
const iframe = document.createElement('iframe')

iframe.id = 'frame'
iframe.style = 'display:none;'
iframe.name = 'polling'
iframe.src = 'target.api' // 这里需要是同域，否则无法获取到iframe内容
iframeCon.appendChild(iframe)

iframe.onload = function () {
  const iloc = iframe.contentWindow.location
  const idoc = iframe.contentDocument
  setTimeout(function () {
    text = idoc.getElementsByTagName('body')[0].textContent
    console.log(text)
    iloc.reload() // 刷新页面，再次获取信息，并且会触发onload函数
  }, 2000)
}
```

### 四. 自适应 iframe
1. 去掉滚动条  
    ```html
    <iframe src="./iframe.html" id="iframe1" scrolling="no"></iframe>
    ```

2. 设置高度  
    ```js
    const iframe = document.getElementById('iframe1')
    const iwindow = iframe.contentWindow
    const idoc = iwindow.document
    iframe.height = idoc.body.offsetHeight
    ```

### 五. iframe 安全性
出现安全性有两个方面：一是你的网页被别人iframe，一是你iframe别人的网页
#### 1. 防嵌套
点击劫持就是使用iframe来拦截click事件。为了防止网站被钓鱼，可以使用 `window.top` 来防止你的网页被iframe
```js
// 限定网页不能被嵌套在任意网页内
if (window !== window.top) {
  window.top.location.href = window.location.href
}

// 限制不同域名嵌套
try {
  if (top.location.hostname !== window.location.hostname) {
    top.location.href = window.location.href
  }
} catch (error) {
  top.location.href = window.location.href
}
```

#### 2. X-Frame-Options
X-Frame-Options 是一个响应头。
```
X-Frame-Options: DENY
拒绝任何iframe的嵌套请求

X-Frame-Options: SAMEORIGIN
iframe页面的地址只能为同源域名下的页面

X-Frame-Options: ALLOW-FROM http://asd.com
可以在指定的origin url的iframe中加载
```

#### 3. CSP
响应头中配置的字段
```
Content-Security-Policy: default-src 'self'
```
可以配置各种指定资源路径
```
default-src,
script-src,
style-src,
img-src,
connect-src,
font-src,
object-src,
sandbox,
child-src,
...
```
其中，child-src用来指定iframe有效加载路径，和 X-Frame-Options 中的 allow-origin 类似。  
sandbox 和 iframe 的 sandbox 属性一样。
```
Content-Security-Policy: child-src 'self' http://example.com; sandbox allow-forms allow-same-origin
```

#### 4. sandbox
给指定 iframe 设置一个沙盒模型限制 iframe 的更多权限（IE10+）
```html
<iframe sandbox src=""></iframe>
```
这样会对 iframe 页面进行一系列的限制：
1. script 脚本不能执行
2. 不能发送 ajax 请求
3. 不能使用本地存储，即 localStorage, cookie等
4. 不能创建新的弹窗和window
5. 不能发送表单
6. 不能加载额外插件比如 flash 等
7. ...

### 六. iframe 跨域
#### 1. 传统方案
1. 以及域名相同，二级域名不同，通过设置 `document.domain` 共享 Cookie
2. 片段识别符  
    父窗口把信息写入子窗口的片段识别符
    ```js
    const src = originURL + '#' + data
    document.getElementById('myIframe').src = src
    ```
    子窗口通过监听 `hashChange` 时间得到通知
3. window.name  

#### 2. postMessage
既可以子传父，也可以父传子，并且可以突破同源限制
```js
// 子页面
window.parent.postMessage('close', *)

// 父页面
window.addEventListener('message', function(event) {
  if (event.data === 'close') {
    console.log('收到，关闭吧')
  }
})
```

### 七. 引用
[iframe, 我们来谈一谈](https://juejin.cn/post/6844903463193673741#heading-1)

[浏览器同源政策及其规避方法](https://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)

[Iframe 有什么好处，有什么坏处？国内还有哪些知名网站仍用Iframe，为什么？有哪些原来用的现在抛弃了？又是为什么？](https://www.zhihu.com/question/20653055)