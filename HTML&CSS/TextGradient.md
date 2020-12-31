### CSS文字渐变色

#### 1. 借助mask-image属性
```html
<div class="box" text="css文字渐变色">css文字渐变色</div>
```
```css
.box {
    color: #333;
    position: relative;
}
.box:before {
    content: attr(text);
    position: absolute;
    z-index: 10;
    color: pink;
    -webkit-mask: linear-gradient(to left, red, transparent);
}
```
#### 2. background-clip + text-fill-color
```css
.box {
    background: linear-gradient(to right, red, blue); // 设置背景为渐变色
    -webkit-background-clip: text;  // 规定背景的绘制区域
    color: transparent; // 文字为透明色，让后面背景色显示出来
}
```