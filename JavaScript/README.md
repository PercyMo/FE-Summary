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

    1. 变量提升
        通常JS引擎会在正式执行之前先进行一次预编译，在这个过程中，首先将变量声明及函数声明提升至当前作用域的顶端，然后进行接下来的处理。   
        例子：
        ```js
        var foo = 3;
        function hoistVariable() {
            var foo = foo || 5;
            console.log(foo); // 5
        }
        hoistVariable();
        ```
        `foo || 5`这个表达式的结果是5而不是3，虽然外层作用域有个foo变量，但函数内是不会去引用的，因为预编译之后的代码逻辑是这样的：
        ```js
        var foo = 3;
        // 预编译之后
        function hoistVariable() {
            var foo;
            foo = foo || 5;
            console.log(foo); // 5
        }
        hoistVariable();
        ```
    2. 函数提升
        ```js
        function hoistFunction() {
            foo(); // 2
            var foo = function() {
                console.log(1);
            };
            foo(); // 1
            function foo() {
                console.log(2);
            }
            foo(); // 1
        }
        hoistFunction();
        ```
        运行后我们会发现，输出的结果依次是2 1 1   
        因为`JavaScript`中的函数是一等公民，函数声明的优先级最高，会被提升至当前作用域最顶端，所以第一次调用时实际执行了下面定义的函数声明，然后第二次调用时，由于前面的函数表达式与之前的函数声明同名，故将其覆盖，以后的调用也将会打印同样的结果。上面的过程经过预编译之后，代码逻辑如下：
        ```js
        // 预编译之后
        function hoistFunction() {
            var foo;
            foo = function foo() {
                console.log(2);
            }
            foo(); // 2
            foo = function() {
                console.log(1);
            };
            foo(); // 1
            foo(); // 1
        }
        hoistFunction();
        ```
    3. 最佳实践
        理解变量提升和函数提升可以使我们更了解这门语言，更好地驾驭它，但是在开发中，我们不应该使用这些技巧，而是要规范我们的代码，做到可读性和可维护性。

        具体的做法是：无论变量还是函数，都必须先声明后使用。

#### 第6章 面向对象的程序设计
3. 继承
    1. **原型对象与原型链（_重点内容_）**
        1. prototype和__proto__的区别

            <img src="https://note.youdao.com/yws/public/resource/0c7e34c500438947463771cc1073655a/xmlnote/WEBRESOURCE247de9dd6ed3d119b222d9620aac2337/899" width="500" />

            ```js
            var a = {};
            a.prototype     // undefined
            a.__proto__     // Object {}

            var b = function() {};
            b.prototype     // b {}
            ```
        2. __proto__属性指向谁

            <img src="https://note.youdao.com/yws/public/resource/0c7e34c500438947463771cc1073655a/xmlnote/WEBRESOURCEf94f0664c6cc8dec86029683129a504c/901" width="500" />

            ```js
            // 1. 字面量方式
            var a = {};
            a.__proto__                                 // Object {}
            a.__Proto__ === a.constructor.prototype;    // true

            // 2. 构造器方式
            var A = function() {};
            var a = new A();
            a.__proto__                                 // A {}
            a.__proto__ === a.constructor.prototype;    // true

            // 3. Object.create()方式
            var a1 = {a: 1};
            var a2 = Object.creat(a1);
            a2.__proto__;                               // Object {a: 1}
            a2.__proto__ === a.constructor.prototype    // false（此处即为图1中的例外情况）
            ```
        3. 什么是原型链

            <img src="https://note.youdao.com/yws/public/resource/0c7e34c500438947463771cc1073655a/xmlnote/WEBRESOURCEb5e2c35f3974cb59743a6b86f2e23410/897" width="500" />

            ```js
            var A = function() {};
            var a = new A();
            a.__proto__;                                // A {} (即构造器function A 的原型对象)
            a.__proto__.__proto__;                      // Object {} (即构造器function Object 的原型对象)
            a.__proto__.__proto__.__proto__;            // null
            ```
            
            <img src="./images/03.png" width="500" />

            图中由相互关联的原型组成的链状结构就是原型链，也就是蓝色的这条线。


    2. 确定原型和实例的关系
        * 通过`instanceof`操作符，用来测试实例与原型链中出现过的构造函数，结果就会返回`true`
            ```js
            instance instanceof Object;         // true
            instance instanceof SuperType;      // true
            ```
        * 使用`isPrototypeOf()`方法。只要是原型链中出现过的原型，都可以说是该原型链所派生出的实例的原型，`isPrototypeOf()`方法会返回`true`
            ```js
            Object.prototype.isPrototypeOf(instance);       // true
            SuperType.prototype.isPrototypeOf(instance);    // true
            ```
    3. 谨慎定义方法

        通过原型链实现继承时，不能使用对象字面量创建原型方法，因为这会重写原型链。
        ```js
        // SubType 继承了 SuperType
        SubType.prototype = new SuperType();
        // 使用对象字面量添加新方法，会导致上一行代码无效
        SubType.prototype = {
            getSubValue: function() {
                return this.subproperty
            },
            someOtherMethod: function() {
                return false;
            }
        }
        ```
        

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