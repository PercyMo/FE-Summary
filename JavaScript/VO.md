### 变量对象详解

#### 1. 变量对象
如果变量与执行上下文相关，那变量自己应该知道它的数据存储在哪里，并且知道如何访问。这种机制称为变量对象(variable object)。

变量对象(缩写为VO)是一个与执行上下文相关的特殊对象，它存储着在上下文中声明的以下内容：
* 变量 (var, 变量声明);
* 函数声明 (FunctionDeclaration, 缩写为FD);
* 函数的形参

我们可以用普通的ECMAScript对象来表示一个变量对象：
```js
VO = {};
```
那么，VO就是执行上下文的属性：
```js
activeExecutionContext = {
  VO: {
    // 上下文数据（var, FD, function arguments)
  }
};
```

#### 2. 不同执行上下文中的变量对象
```
抽象变量对象VO (变量初始化过程的一般行为)
  ║
  ╠══> 全局上下文变量对象GlobalContextVO
  ║        (VO === this === global)
  ║
  ╚══> 函数上下文变量对象FunctionContextVO
           (VO === AO, 并且添加了<arguments>和<formal parameters>)
```
1. 全局上下文中的变量对象

    全局对象是在进入任何执行上下文之前就已经创建了的对象；   
    这个对象只存在一份，它的属性在程序中任何地方都可以访问，全局对象的生命周期终止于程序退出那一刻。

    全局对象初始创建阶段将Math、String、Date、parseInt作为自身属性，等属性初始化，同样也可以有额外创建的其它对象作为属性

2. 函数上下文中的变量对象
    在函数执行上下文中，由活动对象(activation object,缩写为AO)扮演VO的角色。
    ```js
    VO(functionContext) === AO;
    ```
    活动对象是在进入函数上下文时刻被创建的，它通过函数的arguments属性初始化。arguments属性的值是Arguments对象：
    ```js
    AO = {
    arguments: <ArgO>
    };
    ```
    Arguments对象是活动对象的一个属性，它包括如下属性：

    1. callee — 指向当前函数的引用
    2. length — 真正传递的参数个数
    3. properties-indexes (字符串类型的整数) 属性的值就是函数的参数值(按参数列表从左到右排列)。 properties-indexes内部元素的个数等于arguments.length. properties-indexes 的值和实际传递进来的参数之间是共享的。

#### 3. 处理上下文代码的两个阶段
1. 进入执行上下文

    (个人理解：可暂时理解为代码正式执行前的预编译阶段，变量提升和函数提升发生在这个阶段)

    当进入执行上下文(代码执行之前)时，VO里已经包含了下列属性(前面已经说了)：
    * 函数的所有形参(如果我们是在函数执行上下文中)
    * 所有函数声明(FunctionDeclaration, FD)
    * 所有变量声明(var, VariableDeclaration)
    ```js
    function test(a, b) {
        var c = 10;
        function d() {}
        var e = function _e() {};
        (function x() {});
    }
    
    test(10); // call
    ```
    当进入带有参数10的test函数上下文时，AO表现为如下：
    ```js
    AO(test) = {
        a: 10,
        b: undefined,
        c: undefined,
        d: <reference to FunctionDeclaration "d">
        e: undefined
    };
    ```
2. 代码执行
#### 4. 关于变量
任何时候，变量只能通过使用var关键字才能声明。
```js
alert(a); // undefined
alert(b); // "b" 没有声明
 
b = 10;
var a = 20;
```
进入上下文阶段：
```js
VO = {
  a: undefined
};
```
我们可以看到，因为“b”不是一个变量，所以在这个阶段根本就没有“b”，“b”将只在代码执行阶段才会出现(但是在我们这个例子里，还没有到那就已经出错了)。


#### 5. 引用
[变量对象（Variable Object）](http://www.cnblogs.com/TomXu/archive/2012/01/16/2309728.html)