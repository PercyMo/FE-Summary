## 外观模式
### 一. 介绍
#### 1. 定义
为子系统中的一组接口提供一个一致的界面，此模块定义了一个高层接口，这个接口使得子系统更加容易使用。
#### 2. 适用场景
1. 经常用于JS类库(JQuery)中，封装一些接口用于兼容多浏览器
2. 在设计初期，应该要有意识地将不同的两个层分离，比如经典的三层结构，在数据访问层和业务逻辑层、业务逻辑层和表示层之间建立外观Facade
3. 在开发阶段，子系统往往因为不断的重构演化而变得越来越复杂，增加外观Facade可以提供一个简单的接口，减少他们之间的依赖
4. 在维护一个遗留的大型系统时，可能这个系统已经很难维护了，微系统开发一个外观Facade类，为设计粗糙和高度复杂的遗留代码提供比较清晰的接口，让新系统和Facade对象交互，Facade与遗留代码交互所有复杂工作。

#### 3. 优点
1. 可以间接调用子系统，避免因直接访问子系统而产生不必要的错误
2. 易于使用，本身比较轻量

#### 4. 缺点
1. 联系使用时会产生一定的性能问题，因为每次调用时都要检测功能的可用性

### 二. 实现
```js
// DOM 元素事件绑定
function addEvent(element, event, handler) {
  if (element.addEventListener) {
    element.addEventListener(event, handler, false);
  } else if (element.attachEvent) {
    element.attachEvent('on' + event, handler)
  } else {
    element['on' + event] = handler
  }
}
```
用一个接口封装了多个其他接口：
```js
var moblieEvent = {
  // ...
  stop: function(e) {
    e.preventDefault();
    e.stopPropagation();
  }
}
```