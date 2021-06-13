## 单例模式

### 一. 介绍
#### 1. 定义
一个类仅有一个实例，并提供一个访问它的全局访问点（静态方法）

#### 2. 适用场景
* 需要频繁实例化然后销毁的对象
* 创建对象耗时过多或耗资源过多，但又经常用到的对象
* 系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器或资源管理器或者一个弹窗，或者需要考虑资源消耗太大而只允许创建一个对象

#### 3. 优点
* 减少开销（内存、系统性能...）
* 优化和共享资源访问

#### 4. 缺点
* 除一些特殊场景，应避免使用单例模式，因为单例模式会引入全局状态，而一个健康的系统应该避免引入过多的全局状态

### 二. 实现
#### 1. 需要解决的问题
1. 如何确定Class只有一个实例
2. 如何简便地访问Class的唯一实例
3. Class如何控制实例化的过程

#### 2. 实现的关键点
1. 使用IIFE创建局部作用域并立即执行
2. getInstance() 为一个闭包，使用闭包保存局部作用域中的单例对象并返回

#### 3. 代码
1. 使用ES6 class创建单例
```js
class Instance {
  static init() {
    if (!this.instance) {
      this.instance = new Instance()
    }
    return this.instance
  }
}
```

2. 使用闭包创建
```js
const Instance = (function() {
  let instance = null
  const Singleton = function() {}

  return {
    init() {
      if (!instance) {
        instance = new Singleton()
      }
      return instance
    }
  }
})()
```
