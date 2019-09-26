### 四、变量、作用域和内存问题
2. 执行环境及作用域

    JavaScript采用的是词法作用域，函数的作用域在函数定义的时候就决定了。   
    而与词法作用域相对的是动态作用域，函数的作用域是在函数调用的时候才决定的。  
    例子：
    ```js
    // 在node下运行这段js代码
    var value = 1;
    function foo() {
        console.log(value);
    }
    function bar() {
        var value = 2;
        foo();
    }
    bar();

    // 在bash下运行这段bash代码 （bash语言是动态作用域）
    value=1
    function foo () {
        echo $value;
    }
    function bar () {
        local value=2;
        foo;
    }
    bar
    ```
3. [变量对象详解 =>>](./VO.md)

4. [变量提升和函数提升详解 =>>](./Hoisting.md)

5. [执行上下文和执行上下文栈详解 =>>](./Context.md)

### 五、引用类型
2. Array类型
    1. 模拟实现一个forEach
    ```js
    if (!Array.prototype.forEach) {
        Array.prototype.forEach = function(fn, thisObj) {
            var scope = thisObj || window;
            for (var i = 0, len = this.length; i < len; i++) {
                fn.call(scope, this[i], i, this);
            }
        }
    }
    ```
    2. 模拟实现一个filter
    ```js
    if (!Array.prototype.filter) {
        Array.prototype.filter = function(fn, thisObj) {
            var scope = thisObj || window,
                res = [];
            for (var i = 0, len = this.length; i < len; i++) {
                if (fn.call(scope, this[i], i, this)) {
                    res.push(this[i]);
                }
            }
            return res;
        }
    }
    ```

### 六、面向对象的程序设计
3. [原型对象与原型链、继承详解 =>>](./Prototype.md)

4. [设计模式 =>>](./DesignPattern.md)
        
### 七、函数表达式
2. 闭包

    2. [关于this对象详解 =>>](./This.md)
5. [立即执行函数表达式(IIFE)详解 =>>](./IIFE.md)

### 八、BOM
5. [Hash、History & 前端路由模式 =>>](./Router.md)

### 九、DOM
3. [NodeList =>>](./NodeList.md)

### 二十、JSON
1. JSON可以表示三种类型的值：简单值，对象，数组

2. JavaScript字符串和JSON字符串最大的区别在于，JSON字符串必须使用双引号（单引号会导致语法错误）。
    ```json
    "Hello World!"
    ```

3. 于JavaScript对象字面量相比，JSON有三个不一样的地方。
    * 没有声明变量（JSON中没有变量的概念）
    * 末尾没有分号
    * JSON所有对象的属性必须加**双引号**
    ```json
    // JavaScript
    var obj = {
        name: 'Nicholas',
        age: 29
    }

    // JSON
    {
        "name": "Nicholas",
        "age": 29
    }
    ```
4. 何时是JSON，何时不是JSON？

    `{ "prop": "val" }` 这样的声明有可能是JavaScript对象字面量也有可能是JSON字符串，取决于什么上下文使用它，如果是用在string上下文（用单引号或双引号引住，或者从text文件读取）的话，那它就是JSON字符串，如果是用在对象字面量上下文中，那它就是对象字面量。
    ```js
    // 这是JSON字符串
    var foo = '{ "prop": "val" }';
    
    // 这是对象字面量
    var bar = { "prop": "val" };
    ```

5. JSON对象有两个方法：JSON.stringify()和JSON.parse()。分别用于将JSON对象序列化为JSON字符串，和把JSON字符串解析为原生JavaScript值。
    * 值为`undefined`的任何属性都会被跳过

### 二十一、Ajax与Comet
1. [前端跨域问题详解 =>>](./CrossOrigin.md)

### 其他
1. [Event Loop详细解析 =>>](./EventLoop.md)

2. JS单线程与Web Worker

    Html5提出Web Worker标准，允许js创建多个线程，但子线程完全受主线程控制，且不得操作DOM，所以并没有改变js单线程的本质。

3. [架构模式MVC和MVVM =>>](./ArchitecturalPattern.md)

4. [jQuery & zepto =>>](./jq&zepto.md)