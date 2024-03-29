### 三、基本概念
1. [变量和类型 =>>](./Variable&Types.md)

### 四、变量、作用域和内存问题
1. [VO/AO/作用域 =>>](./VO&AO&Scope.md)
2. [变量提升和函数提升详解 =>>](./Hoisting.md)
3. [执行上下文和执行上下文栈详解 =>>](./Context.md)

### 五、引用类型
2. Array类型
    1. [Array数组方法 =>>](./Array.md)

### 六、面向对象的程序设计
3. [原型对象与原型链、继承详解 =>>](./Prototype.md)
        
### 七、函数表达式
1. 闭包
2. [关于this对象详解 =>>](./This.md)
3. [立即执行函数表达式(IIFE)详解 =>>](./IIFE.md)

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

### 二十四、其他
1. [浏览器中Event Loop详细解析 & Node中的事件循环 =>>](./EventLoop.md)

2. JS单线程与Web Worker

    Html5提出Web Worker标准，允许js创建多个线程，但子线程完全受主线程控制，且不得操作DOM，所以并没有改变js单线程的本质。

3. [架构模式MVC和MVVM =>>](./ArchitecturalPattern.md)

4. [jQuery & zepto =>>](./jq&zepto.md)
5. [JS 常用代码片段 =>>](./JSSnippet.md)

#### 1. 函数式编程
1. [函数式编程指北 =>>](./FunctionalProgramming.md)