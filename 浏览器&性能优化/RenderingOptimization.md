## 浏览器渲染优化

### 一. 单个帧的渲染流程——像素管道
![像素管道](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/renderingOptimization/01.jpg?x-oss-process=image/resize,w_450)   

* **JavaScript**。阻塞的发起通常来自JS。每帧的预算约为16.7ms（1000ms / 60帧 ≈ 16.66ms/帧）。但因为浏览器还需要时间将新帧绘制在屏幕上，只有 ~10ms 来执行 JS 代码，过长时间的同步执行 JS 代码肯定会导致超过 10ms 这个阈值。其次，频繁执行一些代码也会过长的占用每帧渲染时间。此外，JS 去获取一些样式还**可能导致强制同步布局**。
* **样式计算(Style)**。此过程是根据匹配选择器（例如`.headline`或`.nav > .nav__item`）计算出哪些元素应用哪些 CSS 规则的过程，这个过程不仅包括计算层叠样式表中的权重来确定样式，也包括内联的样式，来计算每个元素的最终样式。
* **布局(Layout)**。在知道对一个元素应用哪些规则之后，浏览器即可开始计算该元素要占据的空间大小及其在屏幕的位置。网页的布局模式意味着一个元素可能影响其他元素，一般来说如果修改了某个元素的大小或者位置，则需要检查其它所有元素并重排整个页面。
* **绘制(Paint)**。绘制是填充像素的过程。它涉及绘出文本、颜色、图像、边框、阴影，基本上包括元素的每个可视部分。绘制一般是在多个表面（通常称为层）上完成的，绘制包括两个步：1. 创建绘图调用的列表 2. 填充像素。后者也被称作栅格化。
* **合成(Composite)**。由于页面的各个部分有可能被绘制到多个层上，因此它们需要按正确顺序绘制到屏幕上，才能正确渲染页面。尤其对于与其它元素重叠的元素来说。

### 二. 采用更好的CSS方法进行优化
修改不同的样式属性会有以下几种不同的帧流程。  
1. **JS/CSS -> 样式 -> 布局 -> 绘制 -> 合成**  
    ![像素管道](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/renderingOptimization/01.jpg?x-oss-process=image/resize,w_450)  

    如果修改元素的`layout`属性，也就是改变了元素的几何属性（例如宽、高、左侧和顶部位置等），那么浏览器将必须检查其它所有元素，然后自动重排页面。任何受影响的部分都需要重新绘制，而最终绘制的元素需要重新合成。  

2. **JS/CSS -> 样式 -> 绘制 -> 合成**  
    ![像素管道](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/renderingOptimization/02.png?x-oss-process=image/resize,w_450) 

    如果修改`paint only`属性（例如背景图、文字颜色或阴影等），既不会影响页面布局的属性，则浏览器会跳过布局，但仍将执行绘制。

3. **JS/CSS -> 样式 -> 合成**  
    ![像素管道](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/renderingOptimization/03.png?x-oss-process=image/resize,w_450) 

    如果更改一个既不需要布局也不需要绘制的属性，则浏览器将跳到只执行合成。  
    这个最后的版本开销最小，最适合于应用生命周期中的高压力点，例如动画或滚动。  

    我们可以看到 JS、Style 和 Composite 是不可避免的。Layout 和 Paint 这两个步骤不一定会被触发，所以在优化过程中，需要频繁触发的改变，**应该尽可能避免 Layout 和 Paint。**

#### 1. 尽量使用 transform 和 opacity 属性更改来实现动画  
性能最佳的像素管道版本会避免 Layout 和 Paint，为了实现此目标，**需要坚持更改可以由合成器单独处理的属性。常用的两个属性符合条件：transform 和 opacity。**  
想知道每种 CSS 属性是否会触发 Layout，Paint，Composite，可以通过[csstriggers.com](https://csstriggers.com/)查看。

除了 transform 和 opacity，只会触发 composite 的 CSS 属性还有：pointer-events（是否响应鼠标事件）、perspective（透视效果）、perspective-origin（perspective的灭点）、cursor（指针样式）、orphans（设置当元素内部发生分页时必须在页面底部保留的最少行数（用于打印或打印预览））、widows（设置当元素内部发生分页时必须在页面顶部保留的最少行数）

#### 2. 减小选择器匹配的难度  
通过上面几种不同流程的管道图可以发现，只要是修改样式那么必不可少会经过Style：
1. 创建一组匹配选择器，这实质上是浏览器计算出给指定元素应用哪些类、伪类和ID。
2. 从对应的匹配选择器中获取所有样式规则，并计算出此元素的最终样式。

`.fanal-box-title` 和 `.box:nth-last-child(-n+1) .title`，相比复杂的选择器，简单地将选择器与元素匹配开销要小得多，而且嵌套过深的 CSS 选择器依赖了过多的类名，很容易在改动依赖的类名时不小心被影响到。  

使用BEM（个人不是很喜欢），或者CSS Modules，或者 scoped 命名空间等方案。

#### 3. 提升元素到新的层  
有一种能有效减小 Layout 和 Paint 的方法是将元素提升，像 Phontoshop 中层的概念一样，样式也有层的概念，不通过的层根据不同顺序叠加起来，通过 Composite 最终显示出来。在每个层中对这个层进行 Layout 或者 Paint 是不会影响其他层的，一般会根据整个页面的语义将页面分为几个层。  
但是不要滥用层，将每个元素都单独提升到一层，Composite 这个环节有两步，`Update layer Tree` 和 `Composite Layer Tree`，前者负责计算页面中有多少个层，哪些层应该出现并应该按什么顺序叠加起来，后者负责将 layers 合成到屏幕上。层越多，这两个步骤花的时间越长，同时也会占用更多的内存。  
提升元素还有一个好处就是会将动画从 CPU 转移到 GPU 来完成，来实现硬件加速。  
提升元素的两个方法：
```css
.moving-element {
    will-change: transform;
}

.moving-element {
    transform: translateZ(0);
}
```
有些浏览器对`will-change`的支持还不够好，所以一般两个都写上。

#### 4. 使用 flexbox 而不是较早的布局模型  
经过测试，flex布局在现代浏览上相比早期浮动或者定位布局性能更好，而且到现在 flex 布局已经很好的得到了浏览器的支持（IE10除外）。

### 三. 尽量避免 Layout
1. **强制同步重排 - FSL(forced synchronous layout)**  
    布局分为异步布局和同步布局：  
    > 增量布局是异步执行的。Firefox 将增量布局的`reflow 命令`加入队列，而调度程序会触发这些命令的批量执行。Webkit 也有用于执行增量布局的计时器：对呈现树进行遍历，并对 dirty 呈现器进行布局。  
    > 请求样式信息（例如“offsetHeight”）的脚本可同步触发增量布局。  
    > 有时，当初始布局完成之后，如果一些属性（如滚动位置）发生变化，布局就会作为回调而触发。

    除了影响所有呈现器的全局样式更改，例如字体大小和屏幕大小调整的更改都是增量修改，增量修改都是异步的也就给了我们用 thunk 修改的机会。  
    如果我们在js中这样写
    ```js
    let boxes = documents.getElementsByClassName('.box')
    for (let i = 0; i < boxes.lenght; i++) {
        let width = document.getElementById('table')
        boxes[i].style.color = 'red'
    }
    ```
    这种情况下，这一帧相比上一帧布局没有发生改变，那么直接用旧的 Layout 去赋值 width 就可以，不需要对页面进行重排。  
    但是如果这样写：
    ```js
    let boxes = documents.getElementsByClassName('.box')
    for (let i = 0; i < boxes.lenght; i++) {
        let width = document.getElementById('table').width
        boxes[i].style.width = width
    }
    ```
    当下一次循环到来时浏览器还没进重排（因为一直处于 JS阶段）。为了获取正确的 width，浏览器就不得不立刻重新 Layout 获取一个最新值，从而失去了浏览器自身的批量更新的优化，这就是强制同步布局。  
    大多数浏览器通过队列化修改并批量执行来优化重排过程（就是上面说的异步布局），但是如果触发了强制同步布局，每经过一次循环，都会要求浏览器强制刷新队列并要求计划任务立即执行，这就失去了浏览器对重排的优化。 

    [gist](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)中列出了会触发强制同步布局的操作。  

    **避免强制同步布局**  
    1. **使用 `requestAnimationFrame`**,将获取 width 的操作推迟到下一帧，在经过浏览器正常的 Layout 之后，下一帧可以直接拿到 Layout 的值。
        ```js
        requestAnimationFrame(logBoxHeight);

        function logBoxHeight() {
            box.classList.add('super-big');
            console.log(box.offsetHeight);
        }
        ```
    2. **缓存不动变量**，对上面的那个强制同步布局的例子，避免在循环中进行可能会导致强制同步布局的操作
        ```js
        let boxes = documents.getElementsByClassName('.box')
        let width = document.getElementById('table').width
        for (let i = 0; i < boxes.lenght; i++) {
            boxes[i].style.width = width
        }
        ```

2. **FLIP 策略**  
    在做某些动画时，有可能会有连续触发 Layout 步骤的属性，如下图的动画：  
    ![连续触发 Layout 步骤的属性](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/renderingOptimization/04.gif?x-oss-process=image/resize,w_450)  
    
    如果凭直觉来做 ，很可能就是 click 之后加上一个类似于
    ```css
    .element.expanded {
        height: 100%;
        left: 0;
        position: absolute;
        transition: top 200ms ease-in, height 200ms ease-in 50ms;
        top: 0;
        width: 100%;
        z-index: 3;
    }
    ```
    这样的类。但是，可以看到下图中用 Chrome Devtools 打开显示 Paint 区域功能，发现重绘的区域很大，并且伴随着重排，帧数也很低。  
    这时候，就用 transform 来代替对 width 和 height 的改变。  
    其实到这里，就已经可以满足 60 FPS 的效果了，但是为了做到内容与样式分离，将起始与终结的样式全部由 CSS 管理，而中间通过 transform 动画的行为由 CSS 控制，则需要使用 FLIP 方法。  
    **FLIP，F(first)L(last)I(invert)P(play)**  
    * First：在整个动画过程中元素的起始状态
    * Last：在整个动画过程中元素的终止状态
    * Invert：通过 First 和 Last 计算出来的状态，得到一个从 Last 到 First 的变化倍率（比如大小或位置），然后让元素具有终止状态的 class 及刚刚计算出来的 invert state 的 transform 属性，它们两个相抵消，元素在视觉上还是没有任何变化（TODO: 实际上示例的代码中，元素内容被挤压了，这个问题后边再看下，内容过渡是个问题）。比如我们想让一个元素向右移动 10px，再放大两倍那么这个计算出来的相反的 transform 属性就应该是 `transform: translateX(-10px) scale(0.5)`，再给它一个`left: 10px; width: 200px; height: 200px;`（假设原来是`left: 0; width: 100px; height: 100px;`），这两个属性视觉效果上抵消，好像元素从来没有改变过。
    * Play：给元素添加一个 `transition` 效果，再移除元素的 transform 属性，因为此时元素已经是终止状态了，所以就会 transition 到 0，整个过程只有 transform，可以轻松达到 60FPS。  

    核心思想就是 pre-calculation，用代码来表示就是这样：  
    ```js
    var first = el.getBoundingClientRect();

    el.classList.add('totes-at-the-end');

    var last = el.getBoundingClientRect();

    var invert = first.top - last.top;

    el.style.transform = `translateY(${invert}px)`;

    // 这里使用rAF，不用的话
    // el.style.transform = `translateY(${invert}px)`; 和 el.style.transform = ''; 在同一帧中同步执行，就不会有动画效果了。
    requestAnimationFrame(function() {
        el.classList.add('animation-on-transforms');

        el.style.transform = '';
    });

    // 过渡结束后要移除 animation-on-transforms
    el.addEventListener('transitionend', tidyUpAnimations);
    ```

    ![FLIP过程](http://img.vanilla.ink/me/webproject/FE-Summary/Browser/renderingOptimization/05.png?x-oss-process=image/resize,w_450)  

    实际上，FLIP 是将复杂的计算放在了一开始（包括一次强制同步），根据 RAIL 规则，触发后 100ms 的反应时间是可以接受的，所以在 100ms 内完成为止的计算，之后的动画用 `transform` 来达到 60FPS。

    [Demo演示](http://me.vanilla.ink:3002/devtools/optimization.html)

### 四. 高性能的 JavaScript
1. **昂贵的 DOM 操作**  
    其实，JS 的执行速度是很快的，真正慢的是操作 DOM。浏览器通常会将 DOM 和 JS 独立实现，DOM是个与语言无关的 API，但是在浏览器中的接口确是用 JS 来实现的，这意味着通过 JS 去访问另一个模块实现提供的 API 时，会造成很大的开销，这就是造成操作 DOM 慢的原因。

    **批量修改 DOM**  
    在 JS 同步代码中操作（比如添加、删除或者修改尺寸等）DOM 会让浏览器进行重排，包括：
    * 添加或删除可见的 DOM 元素
    * 元素位置改变
    * 元素尺寸改变（包括：宽高、内外边距、边框宽度等属性）
    * 内容改变，例如：文字改变或图片被另一个不同尺寸的图片替代
    * 页面渲染器初始化
    * 浏览器窗口尺寸改变

    其中 Layout 分为全局布局及增量布局，全局布局是指触发了整个呈现树范围的布局，触发原因可能包括：
    * 影响所有呈现器的全局样式更改，例如字体大小更改
    * 屏幕大小调整

    布局可以采用增量方式，也就是只对 dirty 呈现器进行布局（这样可能存在需要额外布局的弊端）。当呈现器为 dirty 时，会异步触发增量布局。例如，当来自网络的额外内容添加到 DOM 树之后，新的呈现器附加到了呈现树中。  
    解决方法是**让 DOM 脱离文档流在对其进行操作，所有操作完成后添加进文档流**，这样可以将重排及重绘的次数降低到一次或两次（脱离文档流及回归文档流的时候），以下方法可以让元素脱离文档流：
    * 隐藏元素 —— `display:none;`（事实上，`display:none;`不会让元素出现在 layout tree 中）
    * 使用 DocumentFragment（**推荐，只有一次 re-flow**）
    * 将原始元素拷贝到一个脱离文档的节点中，修改这个副本，完成之后再替换掉原始元素

2. **事件委托**  
    利用时间冒泡机制来处理子元素事件的绑定，将子元素的 DOM 事件，交由它们的父元素来进行处理，可以有效降低页面的开销 —— 由于事件的冒泡特性，只需要给父节点添加一个监听事件，就能捕获到冒泡自子元素的事件，在通过 `e.target` 来获取真正被操作的子元素。

3. **避免微优化**  
    现在浏览器都有 JIT（Just in time）即时编译的引擎，所以会在运行中编译代码。  
    [知乎](https://www.zhihu.com/question/19672491/answer/13568243)上对 JIT 带来的优化的解释：
    > 动态编译之于静态编译，缺点是它需要即时编译代码，但是有一个优点——编译器可以获得静态编译期所没有的信息。比如：通过运行时的 profilling 可以知道哪些函数是被大量使用的。在哪些 execution path 上哪些函数的参数一直没有变，等等。不要小看这些信息，当即时编译器了解这些信息之后可以在短时间内编译出比静态编译器更优质的二进制码。举例来说，一般程序也遵循90-10原则，即运行时的90%里计算机是在处理其中10%的代码，寻找到这些执行热点代码进行深度优化能得到比静态编译更好的性能。

    所以我们没有必要再去手动地做一些优化，比如在 for 循环中缓存 length，或者像《高性能Javascript》中介绍的 `for (var i = items.length; i--; )` 来减少每次迭代经过的步骤，这在经过 JIT 后是否还会带来优化未可知，但却降低了代码的可读性。  
    举个例子，Redux 中，在执行 subscribe 函数时，用的是 `for (let i = 0; i< listeners.length; i++)`，`listeners.length`本身是可以缓存的（不存在运行过程中 length 改变的情况），但作者给出的理由是 V8 足够智能来做更好的优化。

4. **Web Worker**  
    Web Worker 是提供一种在主线程之外的多线程能力，我们可以将耗时的、阻塞的js操作放在 Web Worker 中，PWA 也是基于 Web Worker 来实现的，并已经成为了前端的未来趋势之一。

5. **使用 requestAnimationFrame**  
    在某一帧当中会被多次触发某个事件（比如scroll），这个事件又会频繁触发样式的修改，导致可能需要多次 Layout 或者 Paint，这是一种浪费，过于频繁的 Layout 和 Paint 会造成卡。这时候就可以用到 rAF。  
    在浏览器的一轮事件循环中，会有 task -> microtask -> UI render 依次执行，rAF 将回调函数放在下一帧的开头，就是让其所在的那一轮的 UI 先 render，然后在下一帧的最开始去执行。

6. **内存管理**  
    JS 是自动管理内存的，浏览器引擎会自动 GC，作为开发者我们无需操心内存（只要保证不出现内存泄漏）。但是 GC 同样是需要消耗时间的（可以从 Chrome Devtools 里的 Performance 看到，GC 需要一段很短的时间），如果数据结构使用不当，造成了内存泄漏或者导致频繁 GC，也会对页面流畅度造成影响。

### 五. 引用
[前端性能优化之浏览器渲染优化 —— 打造60FPS页面](https://github.com/fi3ework/blog/issues/9)


