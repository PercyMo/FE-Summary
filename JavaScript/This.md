## 关于this对象详解

### 一. 前言
当JavaScript代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。

对于每个执行上下文，都有三个重要属性
* 变量对象(Variable object，VO)
* 作用域链(Scope chain)
* this
this与上下文中可执行代码的类型有直接关系，this值在进入上下文时确定，并且在上下文运行期间永久不变。

### 二. this的指向
比较粗暴的理解就是：this 永远指向最后调用它的那个对象

1. 在全局上下文中

    在全局作用域中它的 this 指向当前的全局对象（浏览器端是 Window，node 中是 global）

    ```js
    // 一个小测试

    // 在node命令行中直接运行
    console.log(this)       // global

    // 在node中执行test.js文件
    console.log(this)       // {}
    ```
    在node中一个文件是被当做模块载入执行的，模块中的一些方法与属性可以直接暴露在`module.exports`中，因此，此时js文件中this并非指向全局，而是指向`module.exports`。可以通过下面的代码验证。
    ```js
    console.log(this === module.exports)
    ```
    [这种类似的文章。并不是内容错误，而是没有解释为什么是打印出{}，很容易使人误解以为node下，全局this不指向global](https://segmentfault.com/a/1190000002675091)

2. 函数作为对象方法调用

    在 JavaScript 中，函数也是对象，因此函数可以作为一个对象的属性，此时该函数被称为该对象的方法，在使用这种调用方式时，this 被自然绑定到该对象。
    ```js
    var point = { 
        x : 0,
        y : 0,
        moveTo : function(x, y) { 
            this.x = this.x + x; 
            this.y = this.y + y; 
        }
    }; 
    
    point.moveTo(1, 1)//this 绑定到当前对象，即 point 对象
    ```

3. 作为函数调用

    函数也可以直接被调用，此时 this 绑定到全局对象。   
    如果使用严格模式的话，就是 `undefined`
    ```js
    function makeNoSense(x) {
        this.x = x;
    }

    makeNoSense(5);
    x; // x 已经成为一个值为 5 的全局变量
    ```

    1. 一个特例

    对于内部函数，即声明在另外一个函数体内的函数，这种绑定到全局对象的方式会产生另外一个问题。
    ```js
    var point = {
        x : 0,
        y : 0,
        moveTo : function(x, y) {
            // 内部函数
            var moveX = function(x) {
                this.x = x;//this 绑定到了哪里？
            };
            // 内部函数
            var moveY = function(y) {
                this.y = y;//this 绑定到了哪里？
            };
            
            moveX(x);
            moveY(y);
        }
    };
    point.moveTo(1, 1);
    point.x; //==>0
    point.y; //==>0
    x; //==>1
    y; //==>1
    ```

    ```js
    var name = "windowsName";

    function fn() {
        console.log(this)
        var name = 'Cherry';
        innerFunction();
        function innerFunction() {
            console.log(this, this.name);
        }
    }

    fn()
    ```
    **“当没有明确的执行时的当前对象时，this指向全局对象window。”**

    这属于也 JavaScript 的设计缺陷，正确的设计方式是内部函数的 this 应该绑定到其外层函数对应的对象上，为了规避这一设计缺陷，该变量一般被命名为 that。

4. new关键字

    使用new 的方式实例化构造函数，this 绑定到新创建的对象上。
    ```js
    function Point(x, y){
        this.x = x;
        this.y = y;
    }

    var point = new Point(1, 2);
    ```

### 三. 怎么改变 this 的指向
* 使用 ES6 的箭头函数
* 在函数内部使用 _this = this
* 使用 apply、call、bind
* new 实例化一个对象
```js
var name = "windowsName";

var a = {
    name : "Cherry",

    func1: function() {
        console.log(this.name)     
    },

    func2: function() {
        setTimeout(function() {
            this.func1()
        }, 100);
    }

};

a.func2()
```
在不使用箭头函数的情况下，是会报错的，因为最后调用 setTimeout 的对象是 window，但是在 window 中并没有 func1 函数。

1. 箭头函数
    ```js
    setTimeout(function() {
        this.func1()
    }, 100);

    // 使用箭头函数
    setTimeout(() => {
        this.func1()
    }, 100);
    ```
    箭头函数中没有 this 绑定，必须通过查找作用域链来决定其值，如果箭头函数被非箭头函数包含，则 this 绑定的是最近一层非箭头函数的 this，否则，this 为 `undefined`

2. 在函数内部使用 _this = this

3. 使用 apply、call、bind
    ```js
    // apply
    setTimeout(function() {
        this.func1()
    }.apply(a),100);

    // call
    setTimeout(function() {
        this.func1()
    }.call(a),100);

    // bind
    setTimeout(function() {
        this.func1()
    }.bind(a)(),100);
    ```

### 四. 经典题目
```js
// 输出结果是什么，为什么？
class Logger {
  printName(name = 'world') {
    this.print(`Hello ${name}`);
  }
  print(text) {
    console.log(text);
  }
};

const logger = new Logger();
const  { printName } = logger;
logger.printName(); // Hello world
printName(); // 报错 Uncaught TypeError: Cannot read properties of undefined (reading 'print')
```

```js
var length = 10;
function fn() {
  console.log(this.length);
};
var obj = {
  length: 5,
  method: function (fn) {
    fn(); // 10 无明确调用对象，this 指向 window
    arguments[0](); // 3 输出为 实参长度
  }
};
obj.method(fn, 1, 2); // 改变下这一行，使 fn() 输出 5

// obj.method(fn.bind(obj), 1, 2);
```

### 五. 引用
[深入浅出 JavaScript 中的 this](https://www.ibm.com/developerworks/cn/web/1207_wangqf_jsthis/index.html)

[Javascript中this关键字详解](http://www.cnblogs.com/justany/archive/2012/11/01/the_keyword_this_in_javascript.html)

**进阶**  
[JavaScript深入之从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7)

[深入理解JavaScript系列（13）：This? Yes,this!](http://www.cnblogs.com/TomXu/archive/2012/01/17/2310479.html)