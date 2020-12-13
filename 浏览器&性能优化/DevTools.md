## Chrome DevTools

### 一. Console面板  
组合快捷键  
win：`Ctrl + Shift + J`  
mac：`Command + Option + J`  

![console对象方法](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/01.png?x-oss-process=image/resize,w_400)

1. **`console.clear()`**  
    清空控制台

2. **`console.log(),info(),warn(),error()`**  
    新版chrome的log和info已经看不出区别了
    ```js
    console.log('普通信息')
    console.info('提示性信息')
    console.error('错误信息')
    console.warn('警示信息')
    ```

    ![log,info,warn,error](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/02.png?x-oss-process=image/resize,w_400)

    **使用占位符**  
    ```js
    console.log('今晚%s老虎', '打', '???')
    console.log('今晚%s%c老虎', '打', 'color: red; font-size: 30px;', '???')
    ```
    ![使用占位符](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/03.png?x-oss-process=image/resize,w_200)  

    | 占位符 | 功能 |
    |:- |:- |
    | %s | 字符串 |
    | %d | 整数 |
    | %i | 整数 |
    | %f | 浮点 |
    | %c | css格式字符串 |

3. **`console.time(),timeEnd()`**  
    传入一个参数用于标识起始位置用于统计时间
    ```js
    console.time('t')
    Array(999999).fill({}).forEach((v, index) => v.index = index)
    console.timeEnd('t')
    // t: 26.785888671875 ms
    ```

4. **`console.count()`**  
    计数：可以用于统计某个函数的执行次数
    ```js
    function foo(type = '') {
        type = type ? console.count(type) : console.count()
        return 'type: ' + type
    }
    foo('A')  // A: 1
    foo('B')  // B: 1
    foo()     // default: 1
    foo()     // default: 2
    foo()     // default: 3
    foo('A')  // A: 2
    ```

5. **`console.trace()`**  
    可以追踪代码的调用栈，不用专门去断点看了
    ```js
    console.trace()
    function foo2() {
        console.trace()
    }
    foo2()
    ```

    ![追踪代码的调用栈](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/04.png?x-oss-process=image/resize,w_400)  

6. **`console.table()`**  
    将复合型数据（对象也可以）转为表格显示
    ```js
    var arr = [
        { name: '梅西', qq: 10 },
        { name: 'C罗', qq: 7 },
        { name: '内马尔', qq: 11 },
    ]
    console.table(arr)
    ```

    ![table](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/05.png?x-oss-process=image/resize,w_400)  

7. **`console.dir()`**  
    按便于阅读和打印的形式将对象打印（然而并没觉得更好阅读了）
    ```js
    var obj = {
        name: 'percy',
        age: 26,
        fn: function () {
            alert('percy')
        }
    }
    console.log(obj)
    console.dir(obj)
    ```

    ![对象打印](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/06.png?x-oss-process=image/resize,w_300)  

    打印DOM对象的区别

    ![对象DOM打印](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/07.png?x-oss-process=image/resize,w_400)  

8. **`console.assert()`**  
    断言，用来进行条件判断。当表达式为false时，显示错误信息，不会中断程序执行。
    ```js
    var val = 1
    console.assert(val === 1, '等于 1')
    console.assert(val !== 1, '不等于 1')
    console.log('代码继续执行')
    ```

    ![断言](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/08.png?x-oss-process=image/resize,w_300)  

9. **`console.group(),groupEnd()`**  
    ```js
    console.group('分组 1')
    console.log('分组 1-1111')
    console.log('分组 1-2222')
    console.log('分组 1-3333')
    console.groupEnd()
    console.group('分组 2')
    console.log('分组 2-1111')
    console.log('分组 2-2222')
    console.log('分组 2-3333')
    console.groupEnd()
    ```

    ![分组](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/09.png?x-oss-process=image/resize,w_300)  

10. **选择器**  

    **`$_`**  
    可以记录上次计算的结果，直接用于代码执行：  

    ![选择器$_](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/10.gif?x-oss-process=image/resize,w_400)  

    **`$0,$1...$4`**  
    代表最近审查元素选中过的DOM节点（存储全局变量更加可靠些）  

    ![选择器$0](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/11.gif?x-oss-process=image/resize,w_400)  

    **`$`和`$$`**  
    * `$(selector)`是原生`document.querySelectot()`的封装
    * `$$(selector)`返回的是所有满足选择条件的元素的一个集合，是原生`document.querySelectorAll()`的封装

    **`$x`**  
    将所有匹配的节点放在一个数组里返回
    ```html
    <div>
        <ul>
            <li><p>li下的p1</p></li>
            <li><p>li下的p2</p></li>
            <li><p>li下的p3</p></li>
        </ul>
    </div>
    <p>外面的p</p>
    ```
    ```js
    $x('//li') // 所有的 li
    $x('//p') // 所有的 p
    $x('//li//p') // 所有的 li 下的 p
    $x('//li[p]') // 所有的 li 下的 p
    ```
    ![选择器$x](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/12.png?x-oss-process=image/resize,w_300)  

    **`keys(),values()`**  
    跟ES6对象扩展方法，`object.keys()`和`Object.values()`相同  

    **`copy()`**  
    将变量复制到剪切板：`copy(temp1)`  
    与`save global variable`结合使用神器

### 二. Element面板
组合快捷键  
win：`Ctrl + Shift + C`  
mac：`Command + Option + C`  

1. **css调试**  

    **style**  
    选中目标节点，element面板，查看style->:hov，可以查看对应状态下的样式  
    ![css调试](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/13.png?x-oss-process=image/resize,w_300)  

    **computed**  
    当样式覆盖过多，继承复杂，可以使用computed查看样式  
    ![computed](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/14.png?x-oss-process=image/resize,w_300)  

    **调整某个元素的数值（mac）**  
    `shift + 方向键`：可以 10x 调整单位值
    `alt + 方向键`：可以 100x 调整单位值
    `command + 方向键`：可以 /10 调整单位值

2. **html调试**  
    **隐藏元素**  
    选中节点，直接按键盘H，可以显示/隐藏元素。效果等同于`visibility:hidden`,还是要占据盒模型空间的。（写了一个弹窗，想看下弹窗后页面的情况，这个功能特别好用）

    ![隐藏元素](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/15.gif?x-oss-process=image/resize,w_400)  

    **将某个元素存储到全局临时变量中**  
    选中节点，右键`store as global varible`（在network面板中也能用，尤其是筛选接口的返回值很方便）  

    ![将某个元素存储到全局临时变量中](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/16.gif?x-oss-process=image/resize,w_400)  

    **滚动到某个节点**  
    `Scroll into view`

    **Edge专属3D视图**  

    Chrome没有，控制台 => More tools => 3D View （是挺炫酷的，但说实话感觉用处不大）

3. **DOM断点**  
    可以监听到DOM节点的变更（子节点变动/属性变更/节点移除），并断点至变更DOM状态的js代码行：（当节点并不稳定，可能只出现很短时间，比如拖拽元素的样式，这个断点特别好用）

    ![DOM断点](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/17.gif?x-oss-process=image/resize,w_500)  

### 三. Network面板
1. `Controls` - 控制Network功能选项，以及一些外观展示
2. `Filters` - 控制在Requests Table中显示哪些类型的资源
3. `Overview` - 显示了资源检索时间的时间线。如果看到多条竖线堆叠在一起，则说明这些资源被同时检索
4. `Requests Table` - 此表格列出了检索的每一个资源。 默认情况下，此表格按时间顺序排序，最早的资源在顶部。点击资源的名称可以显示更多信息
5. `Summary` - 可以一目了然看到页面的请求总数、传输的数据总量、加载时间

Network面板从上到下，分为上面几个区域：

#### 1. **(Controls,Filters)区域**  
![(Controls,Filters)区域](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/18.png?x-oss-process=image/resize,w_500)  

* **使用大请求行** - Requests Table的每一个条目显示两行文本字段：主要字段和次要字段
* **捕获屏幕截图** - 将鼠标悬停在某个截图上时，Timeline/Waterfall(时间轴)会显示一条垂直的黄线，指示该帧是合适被捕获的
* **显示概述** - 展示整个生命周期的各个阶段的耗时

#### 2. **Overview区域**  
页面整个生命周期的各个阶段网络资源加载耗时信息的汇总，可以选择区域来筛选`Requests Table` 的详情资源信息。

#### 3. **Requests Table区域**  
* `Name`(名称)：资源的名称
* `Status`(状态)：HTTP状态码
* `Type`(类型)：请求资源的MIME类型
* `Initiator`(发起)：发起请求的对象或类型。有以下几种值：
    * `Parser`(解析器)：Chrome的HTML解析器发起了请求
    * `Redirect`(重定向)：HTTP重定向启动了请求
    * `Script`(脚本)：脚本启动了请求
    * `Other`(其他)：一些其他进程或动作发起了请求，例如用户点击链接跳转页面，或在地址栏中输入网址
* `Size`(大小)：响应头了大小（通常是几百字节）加上响应数据，由服务器提供
* `Time`(时间)：总持续时间，从请求开始到接受响应中的最后一个字节
* `Timeline/Waterfall`(时间轴)：显示所有网络请求的可视化统计信息

> Content-Encoding选项可以用来查看资源的gzip压缩情况

![(Requests Table区域](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/19.png?x-oss-process=image/resize,w_500)  

1. **重新发起XHR请求**  
    除了刷新页面重新请求之外，还可以右键http请求条目，选择`Replay XHR`发起一条新的请求

2. **查看HTTP相关信息**  
    *查看网络请求的参数：* 可以通过点击 `query string parameters` (查询字符串参数)旁边的 `view URL encoded` (查看 URL 编码)或 `view decoded` (查看解码)链接，查看 URL 编码或解码格式的 `query string parameters` (查询字符串参数)。在使用 postman 复制相关入参时尤其实用。
    *查看HTTP响应内容：* 接口的返回值(在 preview 中）同样也可以 `Save global variable` 存储一个全局变量

3. **Size和Time为什么有两行参数**  
    ![Size和Time两行参数](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/20.png?x-oss-process=image/resize,w_500)  
    1. 关于Size列  
        * 第一行表示数据**传输时**的大小，例如上图中的`108KB`
        * 第二行表示的是数据实际的大小`326KB`

        > 在服务端使用`gzip`压缩算法，可以大大提高资源传输效率  
        > **需要注意的点：**  
        > `gzip`压缩只会压缩`响应体`内容，所以适用于返回数据量大的时候，如果数据量太小，有可能会导致数据**传输时**的大小比**实际**大小要大（加入了一些额外的响应头）

    2. 关于Time列  
        * 第一行表示从客户端发送请求到服务器端返回所有数据所花费的总时间，对于上图来说就是`230ms`
        * 第二行表示的是从客户端发送请求到服务器端返回第一个字节所花费的时间，对于上图来说就是`32ms`

        > 第一行的时间代表了所有项目：例如`解析DNS`，`建立连接`，`等待服务器返回数据`，`传输数据`等  
        > 第二行的时间是`总时间 - 数据传输`是时间  

        从上面的分析中我们看到 **从客户端请求到服务器处理结束准备返回数据** 花了`32ms`，但是在进行 **传输数据** 的时候花费了`197ms`  
        对于网慢的用户来说，可能会耗费更长的时间，所以在写代码（接口）的时候，返回的数据量要尽量精简

4. **Waterfall**  
    一条HTTP请求的详细信息：  
    ![Waterfall](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/21.png?x-oss-process=image/resize,w_350)  
    * `Queuing`(排队)：浏览器在以下情况下对请求排队
        1. 存在更高优先级的请求，请求被渲染引擎推迟，这经常发生在images（图像）上，因为它被认为比关键资源（如脚本/样式）的优先级低。
        2. 此源已打开6个TCP连接，达到限值，仅适用于 HTTP/1.0 和 HTTP/1.1。在等待一个即将被释放的不可用的 TCP socket
        3. 浏览器正在短暂分配磁盘缓存中空间，生成磁盘缓存条目（通常非常快）TODO: 这里不很明白，看完浏览器原理再回来看
    * `Stalled`(停滞) - 发送请求之前等待的时间。它可能因为进入队列的任意原因而被阻塞，这个时间包括代理协商的时间。请求可能会因为Queuing中描述的任何原因而停滞。
    * `DNS lookup`(DNS查找) - 浏览器正在解析请求ip地址，页面上每个新域都需要完整地往返（roundtrip）才能进行DNS查找（TODO: 最后一句没懂，或与RTT相关）
    * `Proxy Negotiation` - 浏览器正在与代理服务器协商请求
    * `initial connection`(初始连接) - 建立连接所需要的时间，包括TCP握手/重试和协商SSL
    * `SSL handshake`(SSL握手) - 完成SSL握手所用的时间
    * `Request send`(请求发送) - 发出网络请求所花费是时间，通常是几分之一毫秒
    * `Waiting`(等待 TTFB) - 等待初始响应所花费的时间，也成为`Time To First Byte`(接收到第一个字节所花费的时间)。这个时间除了等待服务器传递响应所花费的时间外，还包括1次往返延迟时间及服务器准备响应所用的时间（服务器发送数据的延迟时间）
    * `Content Download`(内容下载) - 接受响应数据所花费的时间（从接收到第一个字节开始，到下载完最后一个字节结束）
    * `ServiceWorker Preparation` - 浏览器正在启动Service Worker
    * `Request to ServiceWorker` - 正在将请求发送到Service Worker
    * `Receiving Push` - 浏览器正在通过 HTTP/2 服务器推送接收此响应的数据
    * `Reading Push` - 浏览器正在读取之前收到的本地数据

#### 4. **Summary区域**  
![Summary区域](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/22.png?x-oss-process=image/resize,w_500)  

`requests` 查看请求的数量 | `transferred` 请求的总大小 | `resources` 资源 | `Finish` 所有http请求响应完成的时间 | DOMContentLoaded 时间 | load时间  

当页面的初始标记被解析完时，会触发`DOMContentLoaded`，它在Network面板上的显示（页面完全加载的`load`事件，用红线标识）：
* 在 Overview 窗格中的蓝色垂直线表示这个事件
* 在Requests Table 中的蓝色垂直线也表示这个事件
* 在 Summary 窗格中，可以查看事件的确切时间

![DOMContentLoaded 和 load 事件](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/23.png?x-oss-process=image/resize,w_500)  

> DOMContentLoaded 会比 load 时间小，两者时间差大致等于外部资源加载（一般是图片/字体）的时间  
> Finish 时间是页面上所有http请求（不只是XHR请求，还包括DOC，img，js，css等资源是请求）发送到响应完成的时间（如果页面存在一个轮询的接口，这个值也会累加的）  
> 所以 Finish 时间与 DOMContentLoaded 和 load 并无直接关系

#### 5. **使用Network面板进行网络优化**  
1. **排队或停止阻塞**  
    最常见的问题是很多个请求排队或被阻塞。HTTP/1.x 中，chrome限制每个域名下最多同时创建6个TCP长连接，因此资源过多时，就出现了排队现象。解决方案就是：域名分片。  

    ![请求排队或停止阻塞](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/24.png?x-oss-process=image/resize,w_500)  

    > 域名分片不适用于HTTP2，它会影响HTTP2中TCP多路复用的效率。

2. **接收到第一个字节的时间很慢**  

    绿色的块占据比例很高：  

    ![接收到第一个字节的时间很慢](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/25.png?x-oss-process=image/resize,w_250)  

    TTFB建议在200ms以下，以下情况可能导致高TTFB：  
    1. 客户端和服务器间网络条件差
    2. 服务器程序响应很慢

    > 为解决高TTFB，首先排除网络问题。理想情况下（例如接口是node写的），将程序部署在本地，查看TTFB是否依然较高。如果有，那么需要优化应用程序的响应速度。比如优化数据库查询，部分内容实现高速缓存，修改web服务器配置...（让后端改）  
    > 如果本地TTFB低，那可能是客户端和服务器之间的网络问题。

3. **加载缓慢**  

    蓝色的块占据比例很高：  

    ![加载缓慢](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/26.png?x-oss-process=image/resize,w_350)  

    如果 `Content Download` 阶段花费了很多时间，提高服务响应速度、并行下载等优化措施效果并不明显。主要的解决方案是发送更少的字节（比如一张高质量的大图可能几M大小，可以酌情优化下图片的宽高/清晰度）

### 四. Sources面板
#### 1. 自定义代码片段 Snippets  
感觉不太实用

#### 2. 设置断点
1. **断点的面板**  
    ![断点的面板](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/27.png?x-oss-process=image/resize,w_500)  

    ![断点调试按钮](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/28.png?x-oss-process=image/resize,w_500)  

2. **指定位置的中断**(断点调试基本流程)  
    ![断点调试基本流程](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/29.gif?x-oss-process=image/resize,w_500)  

3. **全局事件中断**  
    [sources面板demo](http://me.vanilla.ink:3002/devtools/sources.html)

4. **黑盒模式**  
    把脚本文件放入Blackbox(黑盒)，可以忽略来自第三方库的调用堆栈  
    默认（不开启黑盒）：  
    ![不开启黑盒](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/30.png?x-oss-process=image/resize,w_500)  

    开启黑盒：  
    ![开启黑盒](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/31.png?x-oss-process=image/resize,w_500)  

    1. 打开方式①  
        1. 打开 DevTools Settings (设置)
        2. 在左侧的导航菜单中，单击 Blackboxing (黑箱)
        3. 点击 Add pattern... (添加模式)按钮。
        4. 在 Pattern(模式)文本框输入您希望从调用堆栈中排除的文件名模式。DevTools 会排除该模式匹配的任何脚本。
        5. 在文本字段右侧的下拉菜单中，选择 Blackbox (黑箱)以执行脚本文件但是排除来自调用堆栈的调用，或选择 Disabled (禁用)来阻止文件执行。
        6. 点击Add(添加) 保存

    2. 打开方式②  
        直接在想要忽略的堆栈信息上 blackbox script

5. **DOM断点**  
    查看element面板DOM断点

### 五. Performance面板
> 使用隐身模式打开[demo](http://me.vanilla.ink:3002/devtools/performance.html)  
> 隐身模式可以保证 Chrome 在一个相对干净的环境下运行，假如安装了许多插件，这些插件可能会影响性能分析表现  
> 在 Performance 面板可以查看页面加载过程的详细信息，比如在什么时间开始做什么事情，耗时多久等等。相较于 Network 面板，不仅可以看到通过网络加载资源的信息，还能看到解析 Js、计算样式、重绘等页面加载的方方面面的信息

#### 1. 面板主要区域的划分
1. `Controls` - 开始记录，停止记录和配置记录期间捕获的信息
2. `Overview` - 页面性能的汇总
3. `Flame Chart` - [火焰图（线程面板）]。在火焰图上看到三条（绿色的有好几条）垂直的虚线：
    * 蓝线代表 `DomContentLoaded` 事件
    * 绿线代表首次绘制的时间
    * 红线代表 `load` 事件
4. `Details` - 在 Flame Chart 中，选择了某一事件后，这部分会展示与这个事件相关的更多信息；
    > 如果选择了某一帧，这部分会展示与选中帧相关的信息。如果既没有选中事件也没有选中帧，则这部分会展示当前记录时间段内的相关信息。

![面板主要区域的划分](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/32.png?x-oss-process=image/resize,w_500)

#### 2. 开始记录
> 与台式机和笔记本相比移动设备的CPU功率要小得多。无论何时分析页面，都使用 CPU 限制来模拟页面在移动设备上的表现。确保启用了屏幕截图和内存选项。对于 CPU，选择2倍减速，DevTools 会限制 CPU 使其速度比平时慢 2 倍  
> *注意：如果想要确保在低端设备上运行良好，请将 CPU 设置限制为 20 倍减速。（请告诉我20倍操作按钮在哪里？）*

1. **controls 控制条区域**  
    * 上半区域
        * `Screenshots` 截图：默认勾选，每一帧都会截图
        * `Memory` 内存消耗记录：勾选后可以看到各种内存消耗曲线
    * 下面的 checkbox 区域
        * `Disable javascript samples` [禁用 js 示例]：减少在手机运行时系统的开销，模拟手机运行时开启
        * `Network` [网络模拟 ]：可以模拟在 3G,4G 等网络条件下运行页面
        * `enable anvanced paint instrumentation(slow)` [启用高级画图检测工具（慢速） ]：捕获高级画图检测工具，带来显著的性能开销
        * `CPU` [CPU 限制性能]：主要为了模拟低 CPU 下运行性能

2. **overview 总览区域**  
    ![overview 总览区域](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/33.png?x-oss-process=image/resize,w_500)  

    1. **FPS**  
        绿色竖线越高，FPS越高。FPS图表上的红色块表示长时间帧，该时间段内很可能出现了卡顿。  

        > FPS (frames per second) 是用来分析动画的一个主要性能指标。  
        > Q: 为什么是 60fps？  
        > A: 这和目前大多数显示器的刷新帧率相吻合（60HZ）。这意味着，一秒之内进行60次重新渲染，每次重新渲染的时间不能超过 16.66 毫秒

    2. **CPU**  
        **此面积图指示消耗 CPU 资源的事件类型**。在 CPU 图表中的各种颜色与 `Summary` 面板里的颜色是相互对应的，表示 CPU 在各种事件处理上所花费的时间。如果发现某个处理占用大量时间，那这可能就是找到性能瓶颈的线索。

        | 颜色 | 执行内容 |
        |:- |:- |
        | 蓝色（Loading） | 网络通信和 HTML 解析 |
        | 黄色（Scripting） | Javascript 执行 |
        | 紫色（Rendering） | 样式计算和布局，即重排 |
        | 绿色（Painting） | 更改外观而不会影响布局，重绘 |
        | 灰色（Other） | 其它事件花费的时间 |
        | 白色（Idle） | 空闲时间 |

        > 重排所需的成本比重绘高的多，改变深层次的节点很可能导致父节点的一系列重排

        **单个帧的渲染流程 —— 像素管道**  
        js修改DOM结构或样式 -> 计算style -> layout(重排) -> paint(重绘) -> composite(合成)

    3. **NET**  
        优化网络性能直接使用 network 面板

3. **Flame Chart 火焰图（线程面板）**  
    详细分析某些任务的耗时，从而定位问题。  
    ![Flame Chart 火焰图](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/34.png?x-oss-process=image/resize,w_500)  

    1. **看到几条虚线：**  
        ![线程面板](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/35.gif)  
        * 蓝线代表 `DOMContentLoaded` 事件
        * 绿线代表首次绘制的时间
            * FP(First Paint): 首次绘制
            * FCP(First Contentful Paint): 第一次丰富内容的绘图
            * FMP(First Meaningful Paint): 第一次有意义的 绘图
            * LCP(Largest Contentful Paint): 最大区域内容绘制
        * 红线代表 `load` 时间

        > * `DOMContentLoaded`: 就是DOM内容加载完毕。打开一个网页当输入一个URL，页面的展示首先是空白的，然后过一会，页面会展示出内容，但是页面的有些资源比如图片资源还无法看到，此时页面是可以正常交互的，过一段时间后，图片才完全显示在页面上。从页面空白到展示出页面内容，会触发`DOMContentLoaded`事件。而这段时间就是 HTML 文档被加载和解析完成。
        > * `load` 页面上所有的资源（图片、音频、视频等）被加载以后才会触发 load 事件，简单来说，页面的 load 事件会在`DOMContentLoaded`被触发之后才触发。
    
    2. **Main**展示了主线程运行状况
        * X 轴代表时间，每个长条代表一个 event。长条越长就代表这个 event 花费的时间越长。
        * Y 轴代表调用栈（call stack）。在栈里，上面的 event 调用了下面的 event。

        ![主线程运行状况](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/36.png?x-oss-process=image/resize,w_500)  

        如上图：click 事件触发了 `script_footer_closure.js` 第53行的函数调用。再看下面，Function Call 可以看到一个匿名函数被调用，然后调用 `Me()` 函数，然后调用 `Se()`，以此类推。  

        > DevTools 为脚本分配随机颜色。在上图中，来自一个脚本的函数调用显示为浅绿色。来自另一个脚本的调用被渲染成米色。比较深的黄色表示脚本活动，而紫色的事件表示渲染活动。这些较暗的黄色和紫色事件在所有记录中都是一致的。

        ![主线程事件详情](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/37.gif)  

        1. 在性能报告中有很多数据。可以通过双击，拖动等动作来放大缩小报告范围，从各种时间段来观察分析报告
        2. 在事件长条的右上角，如果出现了红色小三角，说明这个事件是存在问题的，需要特别注意
        3. 双击这个带有红色小三角的事件。在 Summary 会看到详细信息。如果点击了 app.js:94 这个链接，就会跳转到对应的代码处

4. **Details 区域**  
    配合 `Flame Chart` 一起使用
    * `Summary` 区域是一个饼状图总览，汇总了各个事件类型所耗费的总时长，另外还有三个查看选项
    * `Bottom-Up`: 要查看直接花费最多时间的活动时使用
    * `Call Tree`: 想查看导致最多工作的根活动时使用
    * `Event Log`: 想按照记录期间的活动顺序查看活动时使用

### 六. Lighthouse(Audits)面板
懒人专用！（TODO: 但我感觉不是，它的功能好像很强大，后面有时间再探究下！）

![Lighthouse面板](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/38.png?x-oss-process=image/resize,w_500)

* `Performance` 性能
* `Accessibility` 无障碍使用
* `Best Practice` 用户体验
* `SEO` SEO优化
* `Progressive Web App` 页面对于 PWA 的兼容性
### 七. Security面板
用于检测当前页面的安全性

* 如果被请求的页面通过 HTTP 提供服务，那么这个主源就会被标记为不安全
* 如果被请求的页面是通过 HTTPS 获取的，但这个页面接着通过 HTTP 继续从其他来源检索内容，那么这个页面仍然被标记为不安全。

![Lighthouse面板](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/devTools/39.png?x-oss-process=image/resize,w_500)  

### 八. Command
快捷键：`Command + Shift + P`  

截图、CSS/js 覆盖率

### 九. 引用
[Chrome DevTools 全攻略！助力高效开发](https://mp.weixin.qq.com/s/-l5IrY-0CQO_bJe0TWDgDw)