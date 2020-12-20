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

2. **FLIP 策略**  

### 四. 高性能的 JavaScript
1. **昂贵的 DOM 操作**  

2. **事件委托**  

3. **避免微优化**  

4. **Web Worker**  

5. **使用 requestAnimationFrame**  

6. **内存管理**  

### 五. 总结

### 六. 引用
[前端性能优化之浏览器渲染优化 —— 打造60FPS页面](https://github.com/fi3ework/blog/issues/9)


