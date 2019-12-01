## bind、call、apply详解

### 一. bind & call & apply 基础
#### 1. call 和 apply 的区别
他们的作用是是一模一样，只是传参的形式有区别而已。
1. **apply()**  
    `apply` 方法传入两个参数：一个是作为函数上下文的对象，另外一个是作为函数参数所组成的数组。  
    ```js
    var obj = {
        name: 'percy'
    };
    function func(firstName, lastName) {
        console.log(firstName + ' ' + this.name + ' ' + lastName);
    }
    func.apply(obj, ['A', 'B']);    // A percy B
    ```

2. **call()**  
    `call` 方法第一个参数也是作为函数上下文的对象，但是后面传入的是一个参数列表，而不是单个数组。
    ```js
    var obj = {
        name: 'percy'
    };
    function func(firstName, lastName) {
        console.log(firstName + ' ' + this.name + ' ' + lastName);
    }
    func.call(obj, 'C', 'D');    // C percy D
    ```

3. 关于两者的性能  
    `call`的性能一定程度上是要优于`apply`的，这是事实。但引用贺师俊大佬的话：  
    *&nbsp;&nbsp;&nbsp;&nbsp;“请按照你的需求使用，而不是为了所谓性能强行转换成另一种。  要知道如果你自己可以通过转换获得性能提升，没有理由引擎不能做这种优化，只是早晚的事情。  最后，在最新的引擎上两者性能是基本一样的。”*

对于什么时候该用什么方法，其实不用纠结。如果你的参数本来就存在一个数组中，那自然就用`apply`，如果参数比较散乱相互之间没什么关联，就用`call`。

#### 2. call 和 bind 的区别
在 `EcmaScript5` 中扩展了叫 `bind` 的方法，在低版本的 `IE` 中不兼容。它和 `call` 很相似，接受的参数有两部分，第一个参数是是作为函数上下文的对象，第二部分参数是个列表，可以接受多个参数。  

它们之间的区别有以下两点。
1. bind返回值是函数  
    ```js
    var obj = {
        name: 'percy'
    };
    function func() {
        console.log(this.name);
    }

    var func1 = func.bind(obj);
    func1();    // percy
    ```
    `bind`方法不会立即执行，而是返回一个改变了上下文`this`后的函数。而原函数`func`中的`this`并没有被改变，依旧指向全局对象`window`。

2. 参数的使用  
    ```js
    function func(a, b, c) {
        console.log(a, b, c);
    }

    var func1 = func.bind(null, 'percy');

    func('A', 'B', 'C');        // A B C
    func1('A', 'B', 'C');       // percy A B
    func1('B', 'C');            // percy B C
    func.call(null, 'percy');   // percy undefined undefined
    ```
    `call` 是把第二个及以后的参数作为 `func` 方法的实参传进去，而 `func1` 方法的实参实则是在 `bind` 中参数的基础上再往后排。

### 二. call模拟实现
模拟的步骤：
1. 将函数设置为上下文对象的属性
2. 使用eval()执行该函数，并传进参数
3. 删除该函数，并返回执行结果
```js
Function.prototype.myCall = function(context) {
    // 上下文参数 不传 或传 null，这种情况则绑定 window
    var context = context || window;
    // 将函数设置为上下文对象的属性
    context.fn = this;

    var args = [];
    // 从 arguments 对象中取出第2个到最后一个参数，放到数组中
    // arguments是类数组对象，所以可以使用 for 循环
    for (var i = 1; i < arguments.length; i++) {
        args.push('arguments[' + i + ']');
    }
    // 执行后 args 为 ['arguments[1]', 'arguments[2]', 'arguments[3]', ...]

    // args自动调用 Array.toString()
    // 使用 eval() 去执行这个拼好的，并携带了参数的方法 fn
    var result = eval('context.fn(' + args + ')');

    // 从上下文中删除fn属性
    delete context.fn;
    // 返回函数执行结果
    return result;
};
```

### 三. apply模拟实现
```js
Function.prototype.myApply = function(context, arr) {
    context = context || window;
    context.fn = this;

    var result;
    if (!arr) {
        // 若没有第二个参数，则直接执行函数
        result = context.fn();
    } else {
        var args = [];
        // 和 call 的一点小区别，i 是从 0 开始的
        for (var i = 0; i < arr.length; i++) {
            args.push('args[' + i + ']');
        }
        result = eval('context.fn(' + args + ')');
    }
    delete context.fn;
    return result;
};
```

### 四. bind模拟实现
```js
function.bind(thisArgs[, arg1[, arg2[, ...]]])
```
**特点：**
1. 当使用`bind`在`setTimeout`中创建一个函数（作为回调提供）时，作为`thisArgs`传递的任何原始值都将转换为`object`
2. 一个绑定函数也能使用`new`操作符创建对象：这种行为就像把原函数当做构造。提供的`this`值被忽略，同时调用时的参数被提供给模拟函数。
    ```js
    var value = 2;

    var foo = {
        value: 1
    };

    function bar(name, age) {
        this.habit = 'shopping';
        console.log(this.value);
        console.log(name);
        console.log(age);
    }

    bar.prototype.friend = 'kevin';

    var bindFoo = bar.bind(foo, 'daisy');

    var obj = new bindFoo('18');
    // undefined    通过bind绑定的this指向foo已经失效
    // daisy
    // 18
    console.log(obj.habit);
    console.log(obj.friend);
    // shopping     obj中的this此时已经指向了obj
    // kevin
    ```

**bind函数模拟完整实现**
```js
Function.prototype.myBind = function(context) {
    // 调用bind的不是函数，抛错
    if (typeof this !== 'function') {
        throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }
    
    var self = this;
    // 截取第 2 到最后一个参数
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function() {};

    var fBound = function() {
        // 使用全部参数
        var bindArgs = Array.prototype.slice.call(arguments);
        // 通过 instanceof 判断是否通过 new 调用了函数
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    };

    // 通过 FNOP 空函数中转，使 fBound 继承this原型上的属性
    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
};
```

### 五. 引用
[JavaScript 中 apply 、call 的详解](https://github.com/lin-xin/blog/issues/7)

[JavaScript深入之call和apply的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)

[JavaScript深入之bind的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)