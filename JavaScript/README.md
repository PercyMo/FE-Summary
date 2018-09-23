### 《JavaScript》学习笔记
> 基于《JavaScript高级程序设计》第三版目录结构

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