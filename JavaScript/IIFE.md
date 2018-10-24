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
(function(){/* code */}()); //推荐使用这个
(function(){/* code */})(); //这个也可以用
```
#### 3. 特殊例子
【保存闭包的状态】就像当函数通过他们的名字被调用时，参数会被传递，而当函数表达式被立即调用时，参数也会被传递。一个立即调用的函数表达式可以用来锁定值并且有效的保存此时的状态，因为任何定义在一个函数内的函数都可以使用外面函数传递进来的参数和变量（这种关系被叫做闭包）
```js
// 它的运行结果可能并不像你想的那样，因为`i`的值从来没有被锁定。相反的，当setTimeout执行时(循环已经被很好的执行完毕)，因此会打印所有元素的总数，因为这是`i`此时的真实值。
// 5 5 5 5 5
for (var i = 0; i < 5; i++) {
	setTimeout(function() {
		console.log(i);
	}, 200);
}


// 而像下面这样改写，便可以了，因为在IIFE里，`i`值被锁定在了`lockedIndex`里。在循环结束执行时，尽管`i`值的数值是所有元素的总和，但每一次函数表达式被调用时，IIFE里的`lockedIndex`值都是`i`传给它的值,所以正确的值被打印。
// 0 1 2 3 4
for (var i = 0; i < 5; i++) {
	(function function_name(lockedIndex) {
		setTimeout(function() {
			console.log(lockedIndex);
		}, 200);
	})(i);
}
```

#### 4. 优势
立即执行函数一个最显著的优势是就算它没有命名或者说是匿名，函数表达式也可以在没有使用标识符的情况下被立即调用，一个闭包也可以在没有当前变量污染的情况下被使用。

#### 5. 模块模式
模块模式方法不仅相当的厉害而且简单。非常少的代码，你可以有效的利用与方法和属性相关的命名，在一个对象里，组织全部的模块代码即最小化了全局变量的污染也创造了私人变量。
```js
var counter = (function(){
var i = 0;
return {
	get: function(){
		return i;
	},
	set: function(val){
		i = val;
	},
	increment: function(){
		return ++i;
	}
}
}());
counter.get();//0
counter.set(3);
counter.increment();//4
counter.increment();//5

counter.i;//undefined (`i` is not a property of the returned object)
i;//ReferenceError: i is not defined (it only exists inside the closure)
```


#### 6. 引用
[立即执行函数表达式(IIFE)详解](https://segmentfault.com/a/1190000007569312?_ea=1386755)