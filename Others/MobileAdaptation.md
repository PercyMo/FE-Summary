## 移动端适配

### 一. 分辨率
#### 1. 像素
像素即一个小方块，具有特定的位置和颜色。可以作为图片或者电子屏幕的最小组成单位。  

<img src="./images/MobileAdaptation/01.png" width="400" />  

通常所说的两种分辨率：图片分辨率和屏幕分辨率

#### 2. 屏幕分辨率
屏幕分辨率指一个屏幕具体由多少个像素点组成。  
`iPhone XS Max` 和 `iPhone SE`的分辨率分别为`2688 x 1242`和`1136 x 640`。这表示手机分别在垂直和水平上所具有的的像素点数。

#### 3. 图像分辨率
通常是指图片含有的像素数。  
比如一张图片的分辨率为`800 x 400`。这表示图片分别在垂直和水平上所具有的像素点数为`800`和`400`。
同一尺寸的图片，分辨率越高，图片越清晰。  

<img src="./images/MobileAdaptation/02.png" width="500" />  

#### 4. PPI
`PPI(Pixel Per Inch)`：每英寸包括的像素数  
`PPI`可以用于描述屏幕的清晰度以及一张图片的质量。  
例：`iPhone XS Max` 和 `iPhone SE`的`PPI`分别为`458`和`326`，这足以证明前者的屏幕更清晰。

#### 5. DPI
`DPI(Dot Per Inch)`：每英寸包括的点数  
这里的点是一个抽象的单位，它可以是屏幕像素点、图片像素点也可以是打印机的墨点。  
平时你可能会看到使用`DPI`来描述图片和屏幕，这时的`DPI`应该和`PPI`是等价的，`DPI`最常用的是用于描述打印机，表示打印机每英寸可以打印的点数。

### 二. 设备独立像素
`设备独立像素`是相对`物理像素`的一个概念。  

<img src="./images/MobileAdaptation/03.png" width="500" />  

右侧屏幕使用了视网膜屏幕的技术，它的显示结果应该是下面的情况，比如列表的宽度为`300`个像素，那么在一条水平线上，左侧屏幕会用`300`个物理像素去渲染它，而右侧屏幕实际上会用`600`个物理像素去渲染它。  
我们必须用一种单位来同时告诉不同分辨率的手机，它们在界面上显示元素的大小是多少，这个单位就是设备独立像素(`Device Independent Pixels`)简称`DIP`或`DP`。上面我们说，列表的宽度为`300`个像素，实际上我们可以说：**列表的宽度为`300`个设备独立像素**。 

<img src="./images/MobileAdaptation/04.png" width="500" />  

打开`chrome`的开发者工具，我们可以模拟各个手机型号的显示情况，每种型号上面会显示一个尺寸，比如`iPhone X`显示的尺寸是`375x812`，实际`iPhone X`的分辨率会比这高很多，这里显示的就是设备独立像素。

#### 1. 设备像素比
设备像素比`device pixel ratio`简称`dpr`，即物理像素和设备独立像素的比值。  
在`web`中，浏览器为我们提供了`window.devicePixelRatio`来帮助我们获取`dpr`。  
在`css`中，可以使用媒体查询`min-device-pixel-ratio`，区分`dpr`：
```css
@media (-webkit-min-device-pixel-ratio: 2),(min-device-pixel-ratio: 2){ }
```
在`React Native`中，我们也可以使用`PixelRatio.get()`来获取`DPR`。  
**注意**：`iPhone 6、7、8 Plus`的实际物理像素是`1080 x 1920`，在开发者工具中我们可以看到：它的设备独立像素是`414 x 736`，设备像素比为`3`，设备独立像素和设备像素比的乘积并不等于`1080 x 1920`，而是等于`1242 x 2208`。**实际上，手机会自动把`1242 x 2208`个像素点塞进`1080 * 1920`个物理像素点来渲染。**

#### 2. 移动端开发
在`iOS`、`Android`和`React Native`开发中样式单位其实都使用的是设备独立像素。

`iOS`的尺寸单位为`pt`，`Android`的尺寸单位为`dp`，`React Native`中没有指定明确的单位，它们其实都是设备独立像素`dp`。

为了适配所有机型，我们在写样式时需要把物理像素转换为设备独立像素：例如：如果给定一个元素的高度为`200px`(这里的`px`指物理像素，非`CSS`像素)，`iphone6`的设备像素比为`2`，我们给定的`height`应为`200px/2=100dp`。

#### 3. WEB端开发
在写`CSS`时，我们用到最多的单位是`px`，即`CSS`像素，当页面缩放比例为`100%`时，一个`CSS`像素等于一个设备独立像素。  

但是`CSS`像素是很容易被改变的，当用户对浏览器进行了放大，`CSS`像素会被放大，这时一个`CSS`像素会跨越更多的物理像素。  

`页面的缩放系数 = CSS像素 / 设备独立像素。`

#### 4. 关于屏幕
`Retina`屏幕只是苹果提出的一个营销术语：  

比如：给你一块超大尺寸的屏幕，即使它的`PPI`很高，`DPR`也很高，在近距离你能看清它的像素点，这就不算`Retina`屏幕。

<img src="./images/MobileAdaptation/05.png" width="500" />  

我们经常见到用`K`和`P`这个单位来形容屏幕：

`P`代表的就是屏幕纵向的像素个数，`1080P`即纵向有`1080`个像素，分辨率为`1920X1080`的屏幕就属于`1080P`屏幕。 我们平时所说的高清屏其实就是屏幕的物理分辨率达到或超过`1920X1080`的屏幕。  

`K`代表屏幕横向有几个`1024`个像素，一般来讲横向像素超过`2048`就属于`2K`屏，横向像素超过`4096`就属于`4K`屏。

### 三. 视口
#### 1. 布局视口
<img src="./images/MobileAdaptation/06.png" width="400" />  

布局视口是网页布局的基准窗口，在`PC`浏览器上，布局视口就等于当前浏览器的窗口大小（不包括`borders` 、`margins`、滚动条）。  

在移动端，布局视口被赋予一个默认值，大部分为`980px`，这保证`PC`的网页可以在手机浏览器上呈现，但是非常小，用户可以手动对网页进行放大。  

我们可以通过调用`document.documentElement.clientWidth / clientHeight`来获取布局视口大小。

#### 2. 视觉视口
<img src="./images/MobileAdaptation/07.png" width="400" />  

视觉视口(`visual viewport`)：用户通过屏幕真实看到的区域。

视觉视口默认等于当前浏览器的窗口大小（包括滚动条宽度）。  

我们可以通过调用`window.innerWidth / innerHeight`来获取视觉视口大小。

#### 3. 理想视口
<img src="./images/MobileAdaptation/04.png" width="400" />  

在浏览器调试移动端时页面上给定的像素大小就是理想视口大小，它的单位正是设备独立像素。  

上面在介绍`CSS像素`时曾经提到`页面的缩放系数 = CSS像素 / 设备独立像素`，实际上说`页面的缩放系数 = 理想视口宽度 / 视觉视口宽度`更为准确。  

所以，当页面缩放比例为`100%`时，`CSS像素 = 设备独立像素`，`理想视口 = 视觉视口`。
我们可以通过调用`screen.width / height`来获取理想视口大小。

#### 4. Meta viewport
```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no">
```

`Value` | 可能值 | 描述 
:- |:- |:- 
`width` | 正整数或`device-width` | 以`pixels`（像素）为单位， 定义布局视口的宽度。
`height` | 正整数或`device-height` | 以`pixels`（像素）为单位， 定义布局视口的高度。
`initial-scale` | `0.0 - 10.0` | 定义页面初始缩放比率。
`minimum-scale` | `0.0 - 10.0` | 定义缩放的最小值；必须小于或等于`maximum-scale`的值。
`maximum-scale` | `0.0 - 10.0` | 定义缩放的最大值；必须大于或等于`minimum-scale`的值。
`user-scalable` | 一个布尔值（`yes`或者`no`） | 如果设置为 `no`，用户将不能放大或缩小网页。默认值为 `yes`。

#### 5. 移动端适配
`device-width`就等于理想视口的宽度，所以设置`width=device-width`就相当于让布局视口等于理想视口。  

由于`initial-scale = 理想视口宽度 / 视觉视口宽度`，所以我们设置`initial-scale=1;`就相当于让视觉视口等于理想视口。  

这时，`1个CSS像素`就等于`1个设备独立像素`，而且我们也是基于理想视口来进行布局的，所以呈现出来的页面布局在各种设备上都能大致相似。

### 四. 1px问题
在`设备像素比大于1`的屏幕上，我们写的`1px`实际上是被多个物理像素渲染，这就会出现`1px`在有些屏幕上看起来很粗的现象。

#### 1. border-image
基于`media`查询判断不同的设备像素比给定不同的`border-image`：
```css
.border_1px {
    border-bottom: 1px solid #000;
}
@media only screen and (-webkit-min-device-pixel-ratio: 2) {
    .border_1px {
        border-bottom: none;
        border-width: 0 0 1px 0;
        border-image: url(../img/1pxline.png) 0 0 2 0 stretch;
    }
}
```

#### 2. background-image
和`border-image`类似，准备一张符合条件的边框背景图，模拟在背景上。
```css
.border_1px {
    border-bottom: 1px solid #000;
}
@media only screen and (-webkit-min-device-pixel-ratio: 2) {
    .border_1px {
        background: url(../img/1pxline.png) repeat-x left bottom;
        background-size: 100% 1px;
    }
}
```
上面两种都需要单独准备图片，而且圆角不是很好处理，但是可以应对大部分场景。

#### 3. 伪类 + transform（推荐）
```css
.border_1px:before {
    content: '';
    position: absolute;
    top: 0;
    height: 1px;
    width: 100%;
    background-color: #000;
    transform-origin: 50% 0%;
}
@media only screen and (-webkit-min-device-pixel-ratio:2) {
    .border_1px:before{
        transform: scaleY(0.5);
    }
}
@media only screen and (-webkit-min-device-pixel-ratio:3) {
    .border_1px:before{
        transform: scaleY(0.33);
    }
}
```
这种方式可以满足各种场景，如果需要满足圆角，只需要给伪类也加上border-radius即可。
```css
.border_1px:before {
    content: '';
    width: 200%;
    height: 200%;
    position: absolute;
    top: 0;
    left: 0;
    border: 1px solid #000;
    border-radius: 20px;
    transform: scale(0.5, 0.5);
    -webkit-transform-origin: top left;
}
```

#### 4. svg（推荐）
```css
/* PostCSS */
@svg border_1px { 
  height: 2px; 
  @rect { 
    fill: var(--color, black); 
    width: 100%; 
    height: 50%; 
    } 
  } 
.example { border: 1px solid transparent; border-image: svg(border_1px param(--color #00b1ff)) 2 2 stretch; }
```
编译后：
```css
.example { border: 1px solid transparent; border-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' height='2px'%3E%3Crect fill='%2300b1ff' width='100%25' height='50%25'/%3E%3C/svg%3E") 2 2 stretch; }
```

#### 5. 设置viewport
通过设置缩放，让`CSS`像素等于真正的物理像素。  

例如：当设备像素比为3时，我们将页面缩放1/3倍，这时1px等于一个真正的屏幕像素。
```js
const scale = 1 / window.devicePixelRatio;
const viewport = document.querySelector('meta[name="viewport"]');
if (!viewport) {
    viewport = document.createElement('meta');
    viewport.setAttribute('name', 'viewport');
    window.document.head.appendChild(viewport);
}
viewport.setAttribute('content', 'width=device-width,user-scalable=no,initial-scale=' + scale + ',maximum-scale=' + scale + ',minimum-scale=' + scale);
```
实际上，上面这种方案是早先`flexible`采用的方案。

当然，这样做是要付出代价的，这意味着你页面上所有的布局都要按照物理像素来写。这显然是不现实的，这时，我们可以借助`flexible`或`vw`、`vh`来帮助我们进行适配。

### 五. 移动端适配方案
#### 1. flexible方案
核心代码：
```js
function setRemUnit() {
    var docEl = window.document.documentElement;
    var rem = docEl.clientWidth / 10;
    docEl.style.fontSize = rem + 'px';
}
setRemUnit();
```
`rem` 是相对于`html`节点的`font-size`来做计算的。  

上面的代码中，将`html`节点的`font-size`设置为页面`clientWidth`(布局视口)的`1/10`，即`1rem`就等于页面布局视口的`1/10`，这就意味着我们后面使用的`rem`都是按照页面比例来计算的。  

以`iPhone6`为例：布局视口为`375px`，则`1rem = 37.5px`，这时UI给定一个元素的宽为`75px`（设备独立像素），我们只需要将它设置为`75 / 37.5 = 2rem`。  

每个布局都要计算非常繁琐，我们可以借助`PostCSS`的`px2rem`插件来帮助我们完成这个过程。  
安装以下两个包：
* [postcss-pxtorem](https://github.com/cuth/postcss-pxtorem) 是一款 `postcss` 插件，用于将单位转化为 `rem`
* [lib-flexible](https://github.com/amfe/lib-flexible) 用于设置 `rem` 基准值  

```js
// PostCSS 配置 .postcssrc.js
module.exports = {
  plugins: {
    autoprefixer: {
      browsers: ['Android >= 4.0', 'iOS >= 8'],
    },
    'postcss-pxtorem': {
      rootValue: 37.5,
      propList: ['*'],
    },
  }
}
```

#### 2. vw、vh方案
`vh`、`vw`方案即将视觉视口宽度 `window.innerWidth`和视觉视口高度 `window.innerHeight` 等分为 `100` 份。

`vw(Viewport's width)`：`1vw`等于视觉视口的`1%`
`vh(Viewport's height)` :`1vh` 为视觉视口高度的`1%`
`vmin` : `vw` 和 `vh` 中的较小值
`vmax` : 选取 `vw` 和 `vh` 中的较大值

<img src="./images/MobileAdaptation/08.png" width="400" />  

### 六. 适配iphoneX
#### 1. 安全区域
为了适配`iphoneX`为代表的手机的三个改动：圆角（`corners`）、刘海（`sensor housing`）和小黑条（`Home Indicator`）。必须把页面限制在安全范围内，但是不影响整体效果。  

安全区域就是一个不受上面三个效果影响的可视窗口范围。

#### 2. viewport-fit
`viewport-fit`是专门为了适配`iPhoneX`而诞生的一个属性，它用于限制网页如何在安全区域内进行展示。

<img src="./images/MobileAdaptation/09.png" width="400" />  

`contain`: 可视窗口完全包含网页内容  

`cover`：网页内容完全覆盖可视窗口  

默认情况下或者设置为`auto`和`contain`效果相同。

#### 3. env、constant
<img src="./images/MobileAdaptation/10.png" width="400" />  

我们需要将顶部和底部合理的摆放在安全区域内，`iOS11`新增了两个`CSS`函数`env`、`constant`，用于设定安全区域与边界的距离。
函数内部可以是四个常量：

* `safe-area-inset-left`：安全区域距离左边边界距离
* `safe-area-inset-right`：安全区域距离右边边界距离
* `safe-area-inset-top`：安全区域距离顶部边界距离
* `safe-area-inset-bottom`：安全区域距离底部边界距离

注意：我们必须指定`viweport-fit`后才能使用这两个函数：
```html
<meta name="viewport" content="viewport-fit=cover">
```

`constant`在`iOS < 11.2`的版本中生效，`env`在`iOS >= 11.2`的版本中生效，这意味着我们往往要同时设置他们，将页面限制在安全区域内：
```css
body {
    padding-bottom: constant(safe-area-inset-bottom);
    padding-bottom: env(safe-area-inset-bottom);
}
```
当使用底部固定导航栏时，我们要为他们设置`padding`值：
```css
{
    padding-bottom: constant(safe-area-inset-bottom);
    padding-bottom: env(safe-area-inset-bottom);
}
```

### 七. 横屏适配
#### 1. JavaScript检测横屏
```js
window.addEventListener("resize", () => {
    if (window.orientation === 180 || window.orientation === 0) { 
      // 正常方向或屏幕旋转180度
        console.log('竖屏');
    };
    if (window.orientation === 90 || window.orientation === -90 ){ 
       // 屏幕顺时钟旋转90度或屏幕逆时针旋转90度
        console.log('横屏');
    }  
});
```

#### 2. CSS检测横屏
```css
@media screen and (orientation: portrait) {
    /*竖屏...*/
} 
@media screen and (orientation: landscape) {
    /*横屏...*/
}
```

### 七. 图片模糊问题
#### 1. 产生原因
在`dpr > 1`的屏幕上，位图的一个像素可能由多个物理像素来渲染，然而这些物理像素点并不能被准确的分配上对应位图像素的颜色，只能取近似值，所以相同的图片在`dpr > 1`的屏幕上就会模糊:  

<img src="./images/MobileAdaptation/11.png" width="400" />  

#### 2. 解决方案
应该尽可能让一个屏幕像素来渲染一个图片像素，所以，针对不同`DPR`的屏幕，我们需要展示不同分辨率的图片。

如：在`dpr=2`的屏幕上展示两倍图(`@2x`)，在`dpr=3`的屏幕上展示三倍图(`@3x`)。

#### 3. media查询
```css
/* 只适用于背景图 */
.avatar {
    background-image: url(conardLi_1x.png);
}
@media only screen and (-webkit-min-device-pixel-ratio:2) {
    .avatar {
        background-image: url(conardLi_2x.png);
    }
}
@media only screen and (-webkit-min-device-pixel-ratio:3) {
    .avatar {
        background-image: url(conardLi_3x.png);
    }
}
```

#### 4. image-set
```css
/* 只适用于背景图 */
.avatar {
    background-image: -webkit-image-set( "conardLi_1x.png" 1x, "conardLi_2x.png" 2x );
}
```

#### 5. srcset
使用`img`标签的`srcset`属性，浏览器会自动根据像素密度匹配最佳显示图片：
```html
<img src="conardLi_1x.png"
     srcset=" conardLi_2x.png 2x, conardLi_3x.png 3x">
```

#### 6. JS拼接图片url
```js
const dpr = window.devicePixelRatio;
const images =  document.querySelectorAll('img');
images.forEach((img)=>{
    img.src.replace(".", `@${dpr}x.`);
})
```

#### 7. 使用svg
`SVG`的全称是可缩放矢量图，不管放大多少倍都不会失真。

### 八. 引用
[关于移动端适配，你必须要知道的](https://juejin.im/post/5cddf289f265da038f77696c)