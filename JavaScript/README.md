### 《JavaScript》学习笔记
> 基于《JavaScript高级程序设计》第三版目录结构

#### 第4章 变量、作用域和内存问题
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

3. 变量提升和函数提升

    [变量提升和函数提升详解](./Hoisting.md)

#### 第6章 面向对象的程序设计
3. 继承
    [原型对象与原型链、继承详解](./Prototype.md)
        
#### 第7章 函数表达式
5. 立即执行函数表达式(IIFE)
    [立即执行函数表达式(IIFE)详解](./IIFE.md)

#### 第20章 JSON
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

4. JSON对象有两个方法：JSON.stringify()和JSON.parse()。分别用于将JSON对象序列化为JSON字符串，和把JSON字符串解析为原生JavaScript值。
    * 值为`undefined`的任何属性都会被跳过

#### 补充内容
1. Event Loop事件循环

    [Event Loop详细解析](./EventLoop.md)

2. JS单线程与Web Worker

    Html5提出Web Worker标准，允许js创建多个线程，但子线程完全受主线程控制，且不得操作DOM，所以并没有改变js单线程的本质。