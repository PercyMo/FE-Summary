## 300ms点击延迟

### 一. 起因
早期的IOS下，移动端会有双击缩放这个操作，因此浏览器在 click 之后要等待300ms，看用户有没有下一次点击，也就是这次操作是不是双击。

### 二. 解决方案
1. IE/Firefox/Safari(IOS9.3)，可以禁用缩放
```html
<meta name="viewport" content="user-scalable=no">
```
2. chrome32+
```html
<meta name="viewport" content="width=device-width">
```
3. IE10+
```css
html {
    -ms-touch-action: manipulation; /* IE10 */
    touch-action: manipulation; /* IE11+ */
}
```
4. fastclick（使用原生Event） 或者 使用zepto中的tap事件

### 三. 原理
#### 1. click/touch 触发顺序
```js
function log(event) {
    if (event.type === 'touchstart') this._touchstartBeginTime = Date.now()
    console.log(event.type, Date.now() - this._touchstartBeginTime)
}

function listenEvent() {
    const target = document.getElementById('target')
    target.addEventListener('touchstart', log)
    target.addEventListener('touchmove', log)
    target.addEventListener('touchend', log)
    target.addEventListener('mousemove', log)
    target.addEventListener('mousedown', log)
    target.addEventListener('mouseup', log)
    target.addEventListener('click', log)
}
```
当用户点击屏幕时，会依次触发（click事件是在最后触发的）：  
touchstart -> touchmove(0次或多次) -> touchend -> mousemove -> mousedown -> mouseup -> click。

#### 2. fastclick 原理  
在 `touchstart`、`touchmove`、`touchend` 中任意一个（mouse事件不可以）调用 `event.preventDefault`，后续的 `mouse` 事件以及 `click` 事件将不再被触发。  
fastclick 在 `touchend` 阶段调用 `event.preventDefault`，然后通过 `document.createEvent` 创建一个 `MouseEvents`，然后通过 `eventTarget.dispathEvent` 触发对应目标元素上绑定的 click 事件。

### 四. 实现一个 tap 事件
TODO: 实现一个 tap 事件

### 五. 引用
[从移动端click到摇一摇](https://zhuanlan.zhihu.com/p/28052894)