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

1. **(Controls,Filters)区域**  
    ![(Controls,Filters)区域](18)  

    * **使用大请求行** - Requests Table的每一个条目显示两行文本字段：主要字段和次要字段
    * **捕获屏幕截图** - 将鼠标悬停在某个截图上时，Timeline/Waterfall(时间轴)会显示一条垂直的黄线，指示该帧是合适被捕获的
    * **显示概述** - 展示整个生命周期的各个阶段的耗时

2. **Overview区域**  
    页面整个生命周期的各个阶段网络资源加载耗时信息的汇总，可以选择区域来筛选`Requests Table` 的详情资源信息。

3. **Requests Table区域**  
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

    ![(Requests Table区域](19)  

    **重新发起XHR请求**  
    除了刷新页面重新请求之外，还可以右键http请求条目，选择`Replay XHR`发起一条新的请求

    **查看HTTP相关信息**  
    *查看网络请求的参数：* 可以通过点击 `query string parameters` (查询字符串参数)旁边的 `view URL encoded` (查看 URL 编码)或 `view decoded` (查看解码)链接，查看 URL 编码或解码格式的 `query string parameters` (查询字符串参数)。在使用 postman 复制相关入参时尤其实用。
    *查看HTTP响应内容：* 接口的返回值(在 preview 中）同样也可以 `Save global variable` 存储一个全局变量

    **Size和Time为什么有两行参数**  

    **Waterfall**  

4. **Summary区域**  

5. **使用Network面板进行网络优化**  

### 四. Sources面板

### 五. Performance面板

### 六. Lighthouse(Audits)面板

### 七. Security面板

### 八. Command

### 九. 引用
[Chrome DevTools 全攻略！助力高效开发](https://mp.weixin.qq.com/s/-l5IrY-0CQO_bJe0TWDgDw)