### 立即执行函数表达式(IIFE)详解

#### 1. 关于函数调用
在JavaScript里，每个函数，当被调用时，都会创建一个新的执行上下文。因为在函数里定义的变量和函数只能在函数内部被访问，外部无法获取；当调用函数时，函数提供的上下文就提供了一个非常简单的方法创建私有变量。
```js
//因为这个函数的返回值是另一个能访问私有变量i的函数，因此返回的函数实际上被提权(privileged)了
function makeCounter() {
    //i只能从`makeConuter`内部访问
    var i = 0;
    return function(){
        console.log(++i);
    };   
}
//记住：`counter`和`counter2`都有他们自己作用域中的变量 `i`
var counter = makeCounter();
counter();//1
counter();//2

var counter2 = makeCounter();
counter2();//1
counter2();//2

i;//ReferenceError: i is not defined(它只存在于makeCounter里)
```
#### 2. 立即执行函数表达式

最流行的也最被接受的方法是将函数声明包裹在圆括号里来告诉语法分析器去表达一个函数表达式，因为在JavaScript里，圆括号不能包含声明。因为这点，当圆括号为了包裹函数碰上了 function关键词，它便知道将它作为一个函数表达式去解析而不是函数声明。注意理解这里的圆括号和上面的圆括号遇到函数时的表现是不一样的，也就是说。
* 当圆括号出现在匿名函数的末尾想要调用函数时，它会默认将函数当成是函数声明。
* 当圆括号包裹函数时，它会默认将函数作为表达式去解析，而不是函数声明。
```js
//这两种模式都可以被用来立即调用一个函数表达式，利用函数的执行来创造私有变量
(function(){/* code */}()); //Crockford recommends this one
(function(){/* code */})(); //But this one works just as well
```
#### 3. 引用
[立即执行函数表达式(IIFE)详解](https://segmentfault.com/a/1190000007569312?_ea=1386755)