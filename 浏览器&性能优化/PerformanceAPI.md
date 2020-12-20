## Performance API

### 一. 几个实用的API
1. **`performance.now()`方法**  
    `performance.now()`返回`performance.navigationStart`至当前的毫秒数。  
    * `performance.navigationStart`（测量初始点）是浏览器访问最初测量点或者理解为在地址栏输入 URL 后按回车的那一瞬间。
    * 返回值是毫秒数，但带有精确的多位小数。

    用`performance.now()`检测 js 代码的执行时间：
    ```js
    var st = performance.now();
    console.log(Array(9999999).fill(null).filter(v != v).length);
    var end = performance.now();
    console.log(`取值时间${end - st}ms`); // 取值时间 558.7849999992759ms
    ```

2. **`performance.navigation`**  
    `performance.navigation`负责记录用户行为信息，只有两个属性：
    * `redirectCount`：如果有重定向的话，页面通过几次重定向跳转而来
    * `type`：
        * 0 即 `TYPE_NAVIGATENNEXT` 正常进入页面（非刷新，非重定向等）
        * 1 即 `TYPE_RELOAD` 通过 `window.location.reload()`刷新的页面
        * 2 即 `TYPE_BACK_FORWORK`通过浏览器的前进后退按钮进入的页面（历史记录）
        * 255 即 `TYPE_UNDEFINED`非以上方式进入的页面
    
    ```js
    console.log(performance.navigation); // PerformanceNavigation {type: 1, redirectCount: 0}
    ```

3. **`performance.memory`**  
    ```js
    {
        jsHeapSizeLimit: 4294705152,
        totalJSHeapSize: 13841857,
        usedJSHeapSize: 12417637
    }
    ```
    * `usedJSHeapSize`: JS 对象（包括V8引擎内部对象）占用的内存数
    * `totalJSHeapSize`: 可使用的内存
    * `jsHeapSizeLimit`: 内存大小限制

    > 通常，`usedJSHeapSize` 不能大于 `totalJSHeapSize`，否则代表可能出现了内存泄漏。

4. **`performance.getEntires()`**  

5. **`performance.getEntiresByType()`**  

### 二. performance.timing
![timing](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/performanceAPI/01.jpeg)   

TODO: 有个值的注意的地方，在查看微信公众号和jd首页的时候，发现 `domLoading` 时间竟然提前到了 `responseEnd` 之前。不知道什么原因。他们做了什么神奇的优化。
```js
var timing = {
    // 同一个浏览器上一个页面卸载(unload)结束时的时间戳。如果没有上一个页面，这个值会和 fetchStart 相同
    // ^_- 然而，事实情况并不相同！会有几毫秒的时间差
    navigationStart: 1608002060271,

    // 上一个页面 unload 事件抛出时的时间戳。如果没有上一个页面，这个值会返回0
    // 测试了好几次，这一对值都是0，不知道所谓的上一个页面是什么意思
    unloadEventStart: 0,
    // 和 unloadEventStart 相对应，unload 事件处理完成的时间。如果没有上一个页面，这个值会返回0
    unloadEventEnd: 0,

    // 第一个HTTP重定向开始时的时间戳。如果没有重定向，或者重定向中的一个不同源，这个值会返回0
    redirectStart: 0,
    // 最后一个HTTP重定向完成时（也就说是HTTP响应的最后一个比特直接被收到的时间）的时间戳。
    // 如果没有重定向，或者重定向中的一个不同源，这个值会返回0
    redirectEnd: 0,

    // 浏览器准备好使用HTTP请求来获取(fetch)文档的时间戳。这个时间点会在检查任何应用缓存之前。
    fetchStart: 1608002060278,

    // DNS域名查询开始的时间戳
    // 如果使用了持续连接(persistent connection)，或者这个信息存储到了缓存或者本地资源上，则与 fetchStart 值相等
    domainLookupStart: 1608002060313,
    // DNS域名查询完成的时间
    // 如果使用了本地缓存（即无DNS查询）或持久连接，则与 fetchStart 值相等
    domainLookupEnd: 1608002060332,

    // HTTP请求开始向服务器发送时的时间戳。如果使用持久连接，则返回值等同于 fetchStart
    connectStart: 1608002060332,
    // 浏览器与服务器开始安全连接的握手的时间戳。如果当前网页不要求安全连接，则返回0
    secureConnectionStart: 0,
    // 浏览器与服务器之间的连接建立的时间。如果建立的是持久连接，则返回值等同于 fetchStart。连接建立指的是所有握手和认证过程全部结束
    connectEnd: 1608002060348,

    // 浏览器向服务器发出HTTP请求（或开始读取本地缓存时）的时间戳
    requestStart: 1608002060348,
    // 浏览器从服务器收到（或从本地缓存读取）第一个字节的时间戳
    // 如果传输层在开始请求之后失败并且连接被重开，改属性将会被数制成新的请求的相对应的发起时间
    responseStart: 1608002060423,
    // 浏览器从服务器收到（或从本地缓存读取）最后一个字节的时间戳
    // 如果在此之前HTTP连接已经关闭，则返回关闭时的时间戳
    responseEnd: 1608002060424,

    // 当前网页DOM结构开始解析时（即 Document.readyState 属性变为"loading"、相应的readystateChange时间触发时）的时间戳
    domLoading: 1608002060435,
    // DOM结构结束解析、开始加载内嵌资源时（即 Document.readyState 属性变为"interactive"、相应的readystateChange时间触发时）的时间戳
    // 注意只是 DOM 树解析完成，这时候并没有开始加载网页内的资源
    domInteractive: 1608002061536,
    // 当解析器发送 DomContentLoaded 事件，即所有需要被执行的脚本已经被解析时的时间戳
    // 表示 DOM 准备就绪并且没有样式表阻止 js 执行的时间点，这意味着我们可以构建渲染树了，许多 js 框架都会等待此事件发生后，才开始执行它们自己的逻辑
    // 如果没有阻塞解析器的 js，则 domContentLoaded 将在 domInteractive 后立即触发
    domContentLoadedEventStart: 1608002061536,
    domContentLoadedEventEnd: 1608002061536,
    // 表示文档及其所有子资源都准备就绪的时间点（Document.readyState 变为 'complete'、且相对应的readystatechange被触发）
    domComplete: 1608002064494,

    // load 事件被发送的时间点。如果这个事件还未被发送，它的值会是0
    loadEventStart: 1608002064494,
    // 当 load 事件结束，即加载事件完成的时间点。如果这个事件还未被发送，或者尚未完成，它的值会是0
    loadEventEnd: 1608002064494
}
```
通过上面的数据，可以获取到一组非常有用的时间差值：
* 重定向耗时：`redirectEnd - redirectStart`
* DNS查询耗时：`domainLookupEnd - domainLookupStart`
* TCP连接耗时：`connectEnd - connectStart`，包含握手和认证，如果是HTTPS还会有SSL握手
* HTTP请求响应耗时：`responseEnd - requestStart`
* DOM解析加载耗时：`domComplete - domLoading`
* 白屏时间：`domLoading - navigationStart`
* DOMready时间：`domContentLoadedEventEnd - navigationStart`
* onload时间：`loadEventEnd - navigationStart`，也即是onload回调函数执行的时间。

### 四. 引用

[Chrome DevTools 全攻略！助力高效开发](https://mp.weixin.qq.com/s/-l5IrY-0CQO_bJe0TWDgDw)

[腾讯新闻前端团队：深入理解前端性能监控](https://mp.weixin.qq.com/s/p6lzUzzC3KkKdzTu1Woo6w)
