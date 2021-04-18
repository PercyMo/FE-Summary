## 格式化上下文
不同类型的盒子有不同的格式化上下文：
* BFC（Block Formatting Context）块级格式化上下文；
* IFC（Inline Formatting Context）行内格式化上下文；
* FFC（Flex Formatting Context）弹性格式化上下文；
* GFC（Grid Formatting Context）栅格格式化上下文；

### 一. BFC
BFC，"块级格式化上下文"。是页面盒模型布局中的一种 CSS 渲染模式，相当于一个独立的容器，里面的元素和外部的元素相互不影响。

#### 1. 如何创建BFC
* 根元素`<html>`
* 浮动（元素的 `float` 不是 `none`）
* 绝对定位（元素的 `position` 为 `absolute` 或 `fixed`）
* `overflow` 不为 `visiable` 的块元素
* display 为表格布局、弹性布局、网格元素或者行内块元素
* ...[更多创建BFC方式 (MDN)](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)

#### 2. BFC布局规则
* 内部的box会在垂直方向，一个接一个放置（即块级元素独占一行）
* BFC的区域不会与float box重叠（利用这点可以实现自适应两栏布局）
* 内部的Box垂直方向的距离由 `margin` 决定。属于同一个BFC的两个相邻Box的 `margin` 会发生重叠( `margin` 重叠三个条件：**同属于一个BFC；相邻；块级元素**)
* 计算BFC的高度时，浮动元素也参与计算（清除浮动）
* BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素

#### 3. BFC的应用
1. 利用特性4可以解决浮动元素造成的父元素高度塌陷问题
    ```html
    <div class='parent'>
        <div class='float'>浮动元素</div>
    </div>
    ```
    ```css
    .parent {
        overflow:hidden;
    }
    .float {
        float:left;
    }
    ```

2. 阻止相邻元素外边距折叠问题
    ```html
    <style>
    p {
        margin: 100px 0;
    }
    </style>
    <body>
        <p>ABC</p>
        <p>abc</p>
    </body>
    ```
    上面例子中两个P元素之间距离本该为`200px`，然而实际上只有`100px`，因为发生了`margin`重叠。  
    解决方法：**只需要在p外面包裹一层容器，并触发该容器生成一个BFC。那么两个P便不属于同一个BFC，就不会发生margin重叠了**
    ```html
    <style>
    p {
        margin: 100px 0;
    }
    </style>
    <body>
        <p>ABC</p>
        <div class="wrap">
            <p>abc</p>
        </div>
    </body>
    ```

3. 自适应两栏布局
    ```html
    <style>
        .box1 {
            height: 100px;
            width: 100px;
            float: left;
            background: lightblue;
        }
        .box2 {
            width: 200px;
            height: 200px;
            background: #eee;
        }
    </style>
    <body>
        <div class="box1">我是一个左浮动的元素</div>
        <div class="box2">喂喂喂!大家不要生气嘛，生气会犯嗔戒的。悟空你也太调皮了，
        我跟你说过叫你不要乱扔东西，你怎么又……你看，我还没说完你就把棍子给扔掉了!
        月光宝盒是宝物，你把它扔掉会污染环境，要是砸到小朋友怎么办，就算砸不到小朋友，
        砸到花花草草也是不对的。</div>
    </body>
    ```
    ![自适应两栏布局1](http://img.vanilla.ink/me/webproject/FE-Summary/HTML%26CSS/BFC/01.png?x-oss-process=image/resize,w_300)

    上图中，文字围绕着浮动元素排列，不过在这里，这显然不是我们想要的。此时我们可以为 `.box2` 元素的样式加上 `overflow:hidden;` 使其建立一个BFC，让其内容消除对外界浮动元素的影响  

    ![自适应两栏布局2](http://img.vanilla.ink/me/webproject/FE-Summary/HTML%26CSS/BFC/02.png?x-oss-process=image/resize,w_300)

### 二. IFC
块级元素中仅包含内联级别元素，需要注意的是当IFC中有块级元素插入时，会产生两个匿名块将父元素分割开来，产生两个IFC。（所以第一句话中的“仅”一定要连着后边的一起看）

#### 1. IFC渲染规则
* 子元素在水平方向上一个接一个排列，在垂直方向上将以容器顶部开始向下排列
* 子元素无法声明宽高，其中margin/padding在水平方向有效，在垂直方向无效
* 每一行将产生一个匿名行盒（line box），包括该行的所有行内级盒
* 水平方向上，当所有盒的总宽度小于匿名行盒的宽度时，那么水平方向排版由`text-align`属性来决定
* 垂直方向上，行内级盒的对齐方式由`vertical-align`控制，默认对齐为 `baseline`
* IFC 中的行盒高度由`line-height`计算规则来确定，同个IFC下的多个行盒高度可能会不同

#### 2. 渲染分析
```html
<p>It can get <strong>very complicated</storng> once you start looking into it.</p>
```
![](http://img.vanilla.ink/me/webproject/FE-Summary/HTML%26CSS/BFC/03.jpeg?x-oss-process=image/resize,w_500)  

* p标签对内产生一个IFC
* 由于一行无法完全显示，产生了2个行盒；行盒宽度继承了`<p>`的宽度；高度是由里面的内联盒子的`line-height`决定
* It can get：匿名的内联盒子
* very complicated：strong 标签产生的内联盒子
* once you start：匿名的内联盒子
* looking into it.：匿名的内联盒子

#### 3. IFC的应用
1. **水平居中**：当一个块要在环境中水平居中时，设置其为 `inline-block` 则会在外层产生 IFC，通过 `text-align` 则可以使其水平居中  
2. **垂直居中**：创建一个 IFC，用其中一个元素撑开父元素的高度，然后设置其 `vertical-align: middle`，其他行内元素则可以在此父元素下垂直居中  
    一个span和一张图片，默认基线对齐  
    ![](http://img.vanilla.ink/me/webproject/FE-Summary/HTML%26CSS/BFC/04.jpg) 
    ```html 
    <!-- 修改这两个元素的vertical-align -->
    <div>
        <span style="border: 1px solid blue;vertical-align: middle;">我是span</span>
        <img style="border: 1px solid red;vertical-align: middle;" src="favicon.ico" alt="">
    </div>
    ```
    ![](http://img.vanilla.ink/me/webproject/FE-Summary/HTML%26CSS/BFC/05.jpg)  

### 三. FFC
TODO: 补充一下

### 四. GFC

### 5. 引用
[块状格式化上下文BFC](https://github.com/alianggu/blog/issues/6)

[深入理解BFC](https://juejin.im/post/5bc33d0d6fb9a05d1658afc7)