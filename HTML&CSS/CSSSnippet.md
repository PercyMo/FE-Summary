## CSS 常用代码片段

#### - background 背景简写
```css
.box {
    background: url('xxx') no-repeat left top / 100% auto;
}
```

#### - 文本溢出
```css
/* 单行文本溢出 */
.text-overflow {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}

/* 多行文本溢出 */
.text-overflow {
    display: -webkit-box;
    overflow: hidden;
    text-overflow: ellipsis;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
}
```
**元素**较宽不能被完全展示时将其隐藏（常见于标签并排展示，超出时，整个标签隐藏）
```html
<div class="labels">
    <i class="placeholder"></i>
    <span class="label">A really really really really really really really long label</span>
    <span class="label">Cooking</span>
    <span class="label">Coding</span>
    <span class="label">Travel</span>
    <span class="label">Photography</span>
    <span class="label">Reading</span>
</div>
```
```css
.labels {
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-start;
    height: 24px; /* 需要有一个高度 */
    overflow: hidden;
}
.placeholder {
    width: 1px;
    height: 100%;
}
.label {
    flex-shrink: 0;
}
```

#### - CSS 图形
```css
/* 纯css三角箭头 */
.arrow {
    height: 0;
    border-color: transparent #333 transparent transparent;
    border-style: solid;
    border-width: 5px;
}
```

#### - 浏览器特性定制
```css
/* 在谷歌浏览器下设置鼠标选中文字的背景颜色 */
::selection {
    background-color: #f00;
}

/* scrollbar 滚动条样式 仅限webkit平台 */
/* Document scrollbar */
::-webkit-scrollbar {
    width: 8px;
}
::-webkit-scrollbar-track {
    box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
    border-radius: 10px;
}
::-webkit-scrollbar-thumb {
    border-radius: 10px;
    box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.5);
}
/* Scrollable element */
.some-element::webkit-scrollbar {}
```

#### - 清除浮动
```css
/* 现阶段最常见的两种清除浮动的方式，更推荐后者 */
.clearfix:after {
    content: ".";
    height: 0;
    visibility: hidden;
    display: block;
    clear: both;
}
.clearfix { zoom: 1; }
/* 或者 */
.clearfix:before,
.clearfix:after {
    content: ' ';
    display: table;
}
.clearfix:after { clear: both; }
```





## 待整理
```css
/* Box-sizing reset */
 html {
    box-sizing: border-box;
}
*,
*::before,
*::after {
    box-sizing: inherit;
}

/* 单边阴影 */
.box-shadow {
    -webkit-box-shadow: 0 20px 20px -20px red;
    box-shadow: 0 20px 20px -20px red;
}

/* 设置图片灰度 */
.img-filter {
    -webkit-filter: grayscale(100%);
    filter: grayscale(100%);
}

/* 设置文字不可被选中状态 */
.user-select {
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

/* 高dpi图片对于不同设备的适配方案 */
.bg-image {
    background-image: url("$url_@2x.png");
}
@media (-webkit-min-device-pixel-ratio: 3.0), (min-device-pixel-ratio: 3.0) {
    .bg-image {
        background-image: url("$url_@3x.png");
    }
}

/* 移动端设备下的1像素边框 */
.border-1px {
    position: relative;
}
.border-1px:after {
    content: '';
    width: 100%;
    height: 1px;
    display: block;
    position: absolute;
    left: 0;
    bottom: 0;
    border-top: 1px solid rgba(0, 0, 0, .5);
}
@media (-webkit-min-device-pixel-ratio: 1.5), (min-device-pixel-ratio: 1.5) {
    .border-1px:after {
        -webkit-transform: scaleY(0.7);
        transform: scaleY(0.7);
    }
}
@media (-webkit-min-device-pixel-ratio: 2.0), (min-device-pixel-ratio: 2.0) {
    .border-1px:after {
        -webkit-transform: scaleY(0.5);
        transform: scaleY(0.5);
    }
}

/* 标准线性渐变 */
.background-gradient {
    background: -webkit-linear-gradient(90deg, red 50%,orange 60%,yellow 70%,green 80%,blue 90%,indigo 95%,violet);
    background: -o-linear-gradient(90deg, red 50%,orange 60%,yellow 70%,green 80%,blue 90%,indigo 95%,violet);
    background: -moz-linear-gradient(90deg, red 50%,orange 60%,yellow 70%,green 80%,blue 90%,indigo 95%,violet);
    background: linear-gradient(90deg, red 50%,orange 60%,yellow 70%,green 80%,blue 90%,indigo 95%,violet);
}

/* 标准径向渐变 */
.background-gradient {
    background: -webkit-radial-gradient(red, green, blue); /* Safari 5.1 - 6.0 */
    background: -o-radial-gradient(red, green, blue); /* Opera 11.6 - 12.0 */
    background: -moz-radial-gradient(red, green, blue); /* Firefox 3.6 - 15 */
    background: radial-gradient(red, green, blue); /* 标准的语法 */
}
```