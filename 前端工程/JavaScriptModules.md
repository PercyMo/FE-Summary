## JavaScript模块化详解

### 一. 模块化的价值
#### 1. 为什么需要模块化
> 好的书籍会分章节，好的代码得分模块

所谓的模块化，粗俗的讲，就是把一大坨代码，一铲一铲分成一小坨一小坨的代码。当然这种分割也必须是合理的，以便你增减或者修改功能，并且不会影响整体系统的稳定性。

**优势：**
1. **可维护性**，每个模块都是独立的。良好的设计能够极大降低项目的耦合度，以便其能独立于其他的功能被修改。至少维护一个独立的功能模块，要比维护一坨凌乱的代码要容易得多。
2. **减少全局变量污染**，前端开发的初期，我们都在为全局变量头疼，因为会经常触发一些难以排查且非技术性的bug。当一些无关代码不小心重命名了全局变量，我们就会遇到烦人的“命名空间污染”的问题。在模块化规范没有确定之前，其实我们都在极力地避免于此。
3. **可复用性**，前端模块的封装，极大提升了代码的可复用性。从`npm`上找`package`就是个很好的例子。
4. **方便管理依赖关系**，在模块化规范没有完全确定的时候，模块之间相互依赖的关系非常模糊，完全取决于js文件引用的顺序。不仅依赖模糊，且难以维护。

#### 2. 原始的模块化
1. **函数封装**  
    ```js
    // 函数1
    function fn1() {
        // statement
    }
    // 函数2
    function fn2() {
        // statement
    }
    ```
    **优点：**
    * 有一定的功能隔离和封装

    **缺点：**
    * 污染了全局变量，容易引起命名冲突或数据不安全
    * 模块之间关系模糊

2. **对象封装** *（其实就是把变量名塞得深一点）*  
    ```js
    let module1 = {
        data: 'www.baidu.com',
        foo() {
            console.log(`foo() ${this.data}`)
        },
        bar() {
            console.log(`bar() ${this.data}`)
        }
    }

    module1.data = 'other data' // 能直接修改模块内部的数据
    module1.foo()   // foo() other data
    ```
    **优点：**
    * 一定程度上优化了命名冲突，降低了全局变量污染的风险
    * 有一定的模块封装和隔离，并且还可以进一步语义化

    **缺点：**
    * 并没有实质上解决命名冲突的问题
    * 暴露模块成员，内部状态可以被外部改写，数据不安全

3. **IIFE** *（匿名函数闭包）*  
    ```js
    (function(window) {
        let data = 'www.baidu.com'
        function foo() {
            console.log(`foo ${data}`)
        }
        function bar() {
            console.log(`bar ${data}`)
            otherFn()   // 内部调用
        }
        // 内部私有的函数
        function otherFn() {
            console.log('otherFn')
        }
        window.myModule = { foo, bar }
    })(window)

    myModule.foo()  // www.baidu.com
    myModule.bar()
    // www.baidu.com
    // otherFn
    console.log(myModule.data)  // undefined 不能访问模块内部数据
    myModule.data = 'xxx'   // 不是修改模块内部的data
    ```
    基于`IIFE`，还有演进写法  
    引入依赖：
    ```js
    (function(window, $) {
        let data = 'www.baidu.com'
        function foo() {
            console.log(`foo ${data}`)
            $('body').css('background', 'red')
        }
        function bar() {
            console.log(`bar ${data}`)
            otherFn()   // 内部调用
        }
        // 内部私有的函数
        function otherFn() {
            console.log('otherFn')
        }
        window.myModule = { foo, bar }
    })(window, jquery)
    ```
    还有一种所谓的揭示模块模式`Revealing module pattern`
    ```js
    const myModule = (function() {
        let data = 'www.baidu.com'
        function foo() {
            console.log(`foo ${data}`)
        }
        function bar() {
            console.log(`bar ${data}`)
            otherFn()   // 内部调用
        }
        // 内部私有的函数
        function otherFn() {
            console.log('otherFn')
        }
        return {
            foo: foo,
            bar: bar
        }
    })()

    myModule.foo()  // www.baidu.com
    myModule.bar()
    // www.baidu.com
    // otherFn
    ```
    **优点：**
    * 实现了基本的封装
    * 只暴露对外的方法操作，有了`public`和`private`的概念

    **缺点：**
    * 模块依赖关系模糊

### 二. 模块化方案
上述所有解决方案都有一个共同点：使用单一全局变量来把所有的代码包含在一个函数内，由此来创建私有的命名空间和闭包作用域。  
虽然每种方法都比较有效，但也有各自的短板。

#### 1. CommonJS
1. **介绍**  
`CommonJS`主要用于服务端NodeJs中，当然，通过转换打包也可以运行在浏览器端。毕竟服务端加载的模块都是存放于本地磁盘中，所以加载起来比较快，不需要考虑异步方式。  
根据规范，**每一个文件即是一个模块**，其内部定义的变量是属于这个模块的，不会污染全局变量。  
`CommonJS`的核心思想是通过`require`方法来同步加载所依赖的模块，然后通过`exports`或者`module.exports`来导出对外暴露的接口。

2. **特点**  
    * 以文件为一个单元模块，代码运行在模块作用域内，不会污染全局变量
    * 同步加载模块，在服务端直接读取本地磁盘没问题，不太适用于浏览器
    * 模块可以加载多次，但是只会在第一次加载是运行，然后再加载，就是读取的缓存文件。需清理缓存后才可以再次读取缓存文件内容
    * 模块加载的顺序，按照其在代码中出现的顺序
    * 导出的是值的拷贝，这一点和ES6有着很大的不同

3. **基本语法**  
    * 暴露模块：`module.exports = value` 或者 `exports = value`
    * 引入模块：`require(xxx)`，如果是第三方模块，xxx是模块名；如果是自定义模块，xxx是模块文件路径  

    CommonJS规范规定，每个模块内部，module变量代表当前模块。这个变量是一个对象，它的exports属性（即module.exports）是对外的接口。**加载摸个模块，其实是加载该模块的module.exports属性。**
    ```js
    // example.js
    var x = 5;
    var addX = function (value) {
        return value + x;
    };
    module.exports.x = x;
    module.exports.addX = addX;
    ```
    ```js
    var example = require('./example.js');
    console.log(example.x);         // 5
    console.log(example.addX(1));   // 6
    ```

4. **模块的加载机制**  
    **CommonJS模块的加载机制是，输入是被输出的值的拷贝（浅拷贝）。也就是说，一旦输出一个值，模块内部的变化就影响不到这个值**。这点与ES6模块化有重大差异。
    ```js
    // lib.js
    var counter = 3;
    function incCounter () {
        counter++;
    }
    module.exports = {
        counter: counter,
        incCounter: incCounter
    };
    ```
    ```js
    // main.js
    var counter = require('./lib.js').counter;
    var incCounter = require('./lib.js').incCounter;

    console.log(counter);   // 3
    incCounter();
    console.log(counter);   // 3
    ```
    上面代码说明，counter输出以后，lib.js模块内部的变化就影响不到counter了。**这是因为counter是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值**。

5. **在浏览器中使用CommonJS**  
    由于浏览器不支持`CommonJS`规范，因为其根本没有`module`，`exports`，`require`等变量，如果要使用，则必须要转换格式。`Browserify`是目前最常用的`CommonJS`格式转换的工具，我们可以通过安装`browserify`来对其进行转换。但我们仍然需要注意，由于`CommonJS`的规范是阻塞式加载，并且模块文件存放在服务器端，可能会出现假死的等待状态。
    ```sh
    npm install browserify -g
    browserify main.js -o js/bundle/main.js
    ```
    然后在HTML中引入即可使用。

6. **补充知识点**  
    其实在`nodejs`中模块的实现并非完全按照`CommonJS`规范来的，而是进行了取舍。  
    node中一个文件就是一个模块 -> module  
    源码定义如下：
    ```js
    function Module(id = '', parent) {
        this.id = id;
        this.path = path.dirname(id);
        this.exports = {};
        this.parent = parent;
        updateChildren(parent, this, false);
        this.filename = null;
        this.loaded = false;
        this.children = [];
    }
    ```
    ```js
    // 实例化一个模块
    var module = new Module(filename, parent);
    ```
    CommonJS的一个模块，就是一个脚本文件。require命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。
    ```js
    {
        id: '...',
        exports: { ... },
        loaded: true,
        ...
    }
    ```
    上面的代码就是`Node`内部加载模块后生成的一个对象。该对象的`id`属性是模块名，`exports`属性是模块输出的各个接口，`loaded`属性是一个布尔值，表示该模块的脚本是否执行完毕。其它还有很多省略的属性。  
    以后需要用到这个模块时，就会到`exports`属性上面取值。即使再次执行`require`命令，也不会再次执行该模块，而是到缓存之中取值。也就是说，`CommonJS`模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。

#### 2. AMD
Asynchronous Module Definition: 异步模块定义。
解决CommonJS在浏览器端致命的问题：假死。
1. **介绍**  
    `CommonJS`规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。`AMD`规范则是异步加载模块，允许指定回调函数。  
    由于其并非原生js所支持的那种写法，所以使用`AMD`规范开发的时候就需要`require.js`的支持了。  

    `require.js`主要解决两个问题：
    * 异步加载模块
    * 模块之间依赖模糊

2. **特点**
    * 异步加载模块，不会造成因网络问题而出现的假死
    * 显式地列出其依赖关系，并以函数（定义此模块的那个函数）参数的形式将这些依赖进行注入
    * 在模块开始时，加载所以所需依赖

3. **基本语法**  
    1. 定义模块  
        ```js
        define(id, [dependence], callback)
        ```
        * `id`，一个可选参数，说白了就是给模块取个名字，但是却是模块的 唯一标识。如果没有提供，则取脚本的文件名
        * `dependence`，依赖的模块数组
        * `callback`，工厂方法，模块初始化的一些操作。如果是函数，应该只被执行一次。如果是对象，则为模块的输出值

    2. 使用模块  
        ```js
        require([moduleName], callback)
        ```
        * `moduleName`，依赖的模块数组
        * `callback`，依赖模块加载成功之后执行的回调函数（前端异步的通用解决方案）

    3. `data-main`  
        ```html
        <script src="scripts/require.js" data-main="scripts/app.js"></script>
        ```
        `data-main`指定入口文件，比如这里指定`scripts`下的`app.js`文件，那么只有直接或者间接与`app.js`有依赖关系的模块才会被插入到`html`中。

    4. `require.config`  
        通过这个函数可以对`requirejs`进行灵活的配置，其参数为一个配置对象，配置项及含义如下：
        * `baseUrl`，用于加载模块的根路径
        * `paths`，用于映射不存在根路径下面的模块路径
        * `shims`，配置在脚本/模块外面并没有使用`RequireJs`的函数依赖并且初始化函数。假设`underscore`并没有使用`RequireJs`定义，但是你还是想通过`RequireJs`来使用他，那么你就需要在配置中把它定义为一个`shim`
        * `deps`，加载依赖关系数组
        ```js
        require.config({
            // 默认情况下从这个文件开始拉取资源
            baseUrl: 'scripts/app',
            // 如果你的依赖以pb开头，会从scripts/pb加载模块
            paths: {
                pb: '../pb'
            },
            // load backbone as a shim，就是将没有使用requirejs方式定义
            // 模块的东西转变为requirejs模块
            shim: {
                'backbone': {
                    deps: ['underscore'],
                    exports: 'Backbone'
                }
            }
        })
        ```

4. **Demo演示**  
    1. 目录结构  
        ```
        ├── index.html
        └── js
            ├── libs
            │   └── require.js
            ├── main.js
            └── modules
                ├── article.js
                └── user.js
        ```
    2. 页面  
        ```html
        <!-- index.html -->
        <!DOCTYPE html>
        <html>
        <head>
            <title>Modular Demo</title>
        </head>
        <body>
            <!-- 引入require.js并指定js主文件的入口 -->
            <script data-main="js/main" src="js/libs/require.js"></script>
        </body>
        </html>
        ```
    3. 模块定义  
        ```js
        // main.js
        (function () {
            require.config({
                baseUrl: 'js/',
                paths: {
                    article: 'modules/article',
                    user: 'modules/user',
                    // 第三方库模块
                    jquery: 'libs/jquery-1.10.1'    // 注意：写成jQuery会报错
                }
            })

            require(['article'], function (alerter) {
                alerter.consoleMsg()
            })
        })()
        ```
        ```js
        // article.js
        // 定义有依赖的模块
        define(['user'], function (user) {
            const name = 'THE LAST TIME'

            function consoleMsg() {
                console.log(`${name} by ${user.getAuthor()}`);
            }
            
            // 暴露模块
            return { consoleMsg }
        })
        ```
        ```js
        // user.js
        // 定义没有依赖的模块
        define(function () {
            const author = 'Percy'

            function getAuthor() {
                return author.toUpperCase()
            }

            return { getAuthor }  // 暴露模块
        })
        ```

#### 3. CMD
1. **介绍**  
    CMD是阿里的玉伯提出来的，`sea.js`，它和AMD非常相似，文件即为模块，但是其最主要的区别是实现了按需加载。推崇依赖就近原则，模块延迟执行，而AMD所依赖模块是提前执行（`requireJS 2.0` 后也改为了延迟执行）。
    ```js
    // AMD
    define(['./a', './b'], function(a, b) {
        // 依赖一开始就写好
        a.test()
        b.test()
    })

    // CMD
    define(function(require, exports, module) {
        // 依赖可以就近书写
        var a = require('./a')
        a.test()

        // 按需加载
        if (status) {
            var b = require('/b')
            b.test()
        }
    })
    ```
    准确的说，`CMD`是`SeaJS`在推广过程中对模块定义的规范化产物。  
    也可以说`SeaJS`是一个遵循`CMD`规范的`JavaScript`模块加载框架，可以实现`JavaScript`的`CMD`模块化开发方式。  
    `SeaJS`只是实现`JavaScript`的模块化和按需加载，并未扩展`JavaScript`语言本身。`SeaJS`的主要目的是让开发人员更加专注于代码本身，从繁重的`JavaScript`文件以及对象依赖处理中解放出来。  
    `SeaJS`追求简单、自然的代码书写和组织方式，具有如下核心特性：
    * **简单友好的模块定义规范**：`SeaJS`遵循`CMD`规范，可以像`Node.js`一般书写模块代码。
    * **自然直观的代码组织方式**：依赖的自动加载，配置简洁清晰。
    `SeaJS`还提供常用插件，非常有助于开发调试和性能优化，并具有丰富的可扩展接口。

2. **Demo演示**  
    1. 目录结构  
        ```
        ├── index.html
        └── js
            ├── libs
            │   ├── jquery
            │   │   └── jquery
            │   │       └── 1.10.1
            │   │           └── jquery.js
            │   └── sea.js
            ├── main.js
            └── modules
                ├── module1.js
                ├── module2.js
                ├── module3.js
                └── module4.js
        ```
    2. 页面  
        ```html
        <!DOCTYPE html>
        <html>
        <head>
            <title>CMD Demo</title>
        </head>
        <body>
            <!-- sea.js并指定js主文件的入口 -->
            <script src="js/libs/sea.js"></script>
            <script>
                // seajs 的简单配置
                seajs.config({
                    base: './js/libs/',
                    alias: {
                        // 文件路径必须这么写，嵌套多层
                        // jquery源码中对cmd处理的代码，写死了路径
                        // "function" == typeof define && define("jquery/jquery/1.10.1/jquery", [], function() {})
                        'jquery': 'jquery/jquery/1.10.1/jquery.js'
                    }
                })
                // 加载入口模块
                seajs.use('./js/main')
            </script>
        </body>
        </html>
        ```
    3. 模块定义  
        ```js
        // main.js文件
        define(function (require) {
            var m1 = require('./modules/module1')
            var m4 = require('./modules/module4')
            m1.show()
            m4.show()
        })
        ```
        ```js
        // module1.js
        define(function (require, exports, module) {
            var $ = require('jquery')
            // 内部变量数据
            var data = $('title')[0].text
            // 内部函数
            function show() {
                console.log('module1 show() ' + data)
            }
            // 向外暴露
            exports.show = show
        })
        ```
        ```js
        // module2.js
        define(function (require, exports, module) {
            module.exports = {
                msg: 'I Will Back'
            }
        })
        ```
        ```js
        // module3.js
        define(function (require, exports, module) {
            const API_KEY = 'abc123'
            exports.API_KEY = API_KEY
        })
        ```
        ```js
        // module4.js
        define(function (require, exports, module) {
            // 引入依赖模块(异步)
            require.async('./module3', function (m3) {
                console.log('异步引入依赖模块3  ' + m3.API_KEY)
            })
            
            // 引入依赖模块(同步)
            var module2 = require('./module2')
            function show() {
                console.log('module4 show() ' + module2.msg)
            }
            exports.show = show
        })
        ```

#### 4. UMD
1. **介绍**  
    `UMD`是`AMD`和`CommonJS`的综合产物。如上所说，`AMD`的用武之地是浏览器，非阻塞式加载。`CommonJS`主要用于服务端`Nodejs`中使用。所以人们就想到了一个通用的模式`UMD`（universal module definition）。来解决跨平台的问题。  
    没错！就是 `if else` 的写法。  
    核心思想是：先判断是否支持`NodeJs`的模块（`exports`）是否存在，存在则使用`NodeJs`模块模式。  
    再判断是否支持`AMD`（`define`是否存在），存在则使用`AMD`方式加载模块。

2. **常规用法**  
    ```js
    (function (window, factory) {
        if (typeof exports === 'object') {
            module.exports = factory();
        } else if (typeof define === 'function' && define.amd) {
            define(factory);
        } else {
            window.eventUtils = factory();
        }
    })(this, function () {
        // module ...
    })
    ```

#### 5. ES6
1. **介绍**  
在`ES6`中，我们可以使用`import`引入模块，使用`export`导出模块，功能较之于前几个方案更为强大，也是我们所推崇的，但是由于`ES6`目前无法在所有浏览器中执行，所以，我们还需要通过`babel`将不被支持的`import`编译为当前收到广泛支持的`require`。  
`ES6`模块化汲取了`CommonJS`和`AMD`的优点，拥有简介的语法和异步的支持。并且写法也和`CommonJS`非常相似。

2. **特点**  
    * 每一个模块加载多次，JS只执行一次，如果下次再去加载同目录下同文件，直接从内存中读取。一个模块就是一个单例，或者说就是一个对象。
    * 代码是在模块作用域中运行，而不是在全局作用域中运行。模块内部的顶层变量，外部不可见，不会污染全局作用域
    * 模块脚本自动采用严格模式，不管有没有声明`use strict`
    * 模块之中，可以使用`import`命令加载其他模块（`.js`后缀不可省略，需要提供绝对或者相对URL），也可以使用`export`命令输出对外接口
    * 模块之中，顶层的`this`关键字返回`undefined`，而不是指向`window`。也就是说，在模块顶层使用`this`关键字，是无意义的

3. **与CommonJS的差异**  
    两大差异：
    * **CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用**
    * **CommonJS 模块是运行时加载，ES6 模块是编译时输出接口**

    1. 值拷贝 & 值引用  
        ```js
        // counter.js
        var counter = 1;
        function increment() {
            counter++;
        }

        module.exports = {
            counter: counter,
            increment: increment
        }
        ```
        ```js
        // main.js
        var counter = require('./counter');

        counter.increment();
        console.log(counter.counter);
        ```
        `main.js`中的实例是原本模块的拷贝，这就解释了为什么调用`counter.increment()`之后仍然返回`1`。  
        而通过import语句，可以引入实时**只读**的模块。
        ```js
        // counter.js
        let counter = 1;
        function increment() {
            counter++;
        }

        export {
            counter,
            increment
        }
        ```
        ```js
        // main.js
        import * as counter from './counter'

        counter.increment();
        // counter = {};
        // SyntaxError: main.js: "counter" is read-only
        console.log(counter.counter);
        ```

    2. 加载 & 编译  
        因为`CommonJS`加载的是一个对象（`module.exports`），对象只有在脚本运行时才能生成。而`ES6`模块不是一个对象，只是一个静态的定义。在代码解析阶段就会生成。  
        `ES6`模块是编译时输出接口，因此有如下2个特点：
        * `import` 命令会被JS引擎静态分析，优先于模块内的其它内容执行
        * `export` 命令会有变量声明提升的效果，所以 `import` 和 `export` 命令在模块中的位置并不影响程序的输出。
        ```js
        // a.js
        console.log('a.js')
        import { foo } from'./b'
        
        // b.js
        export let foo = 1
        console.log('b.js 先执行')

        // 执行结果:
        //    b.js 先执行
        //    a.js
        ```
        ```js
        // a.js
        import { foo } from './b'
        console.log('a.js')

        export const x = 1
        export const y = () => {
            console.log('y')
        }
        export function z() {
            console.log('z')
        }

        // b.js
        export const foo = 1
        import * as a from './a'
        console.log(a)
        // TODO: 这里存在异议，个人觉得会报错，在浏览器中验证也是会报错。b.js执行第二行时，x,y均未定义（z有定义），* as a 将它们赋值给a的属性，个人觉得无法成立
        // 执行结果:
        // { x: undefined, y: undefined, z: [Function: z] }
        // a.js
        ```
    
    3. 循环加载的差异  
        “循环加载”指的是，a脚本的执行依赖b脚本，而b脚本的执行又依赖a脚本。
        ```js
        // a.js
        var b = require('./b');

        // b.js
        var a = require('./a');
        ```
        循环加载如果处理不好，还可能导致递归加载，使程序无法执行，因此应该避免出现。  
        1. 在CommonJS中，脚本代码在require的时候，就会全部执行。**一旦出现某个模板被“循环加载”，就只输出已经执行的部分，还未执行的部分不会输出。**  
            ```js
            // main.js
            var a = require('./a.js');
            var b = require('./b.js');
            console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);

            // a.js
            exports.done = false;
            var b = require('./b.js');
            console.log('在 a.js 之中，b.done = %j', b.done);
            exports.done = true;
            console.log('a.js 执行完毕');

            // b.js
            exports.done = false;
            var a = require('./a.js');
            console.log('在 b.js 之中，a.done = %j', a.done);
            exports.done = true;
            console.log('b.js 执行完毕');
            ```
            输出结果为：
            ```
            在 b.js 之中，a.done = false
            b.js 执行完毕
            在 a.js 之中，b.done = true
            a.js 执行完毕
            在 main.js 之中, a.done=true, b.done=true
            ```
            从上面我们可以看出：
            * 在`b.js`中，`a.js`没有执行完毕，只执行了第一行
            * main.js执行到第二行时，不会再次执行`b.js`，而是输出缓存的`b.js`的执行结果，即它的第四行

        2. `ES6`处理循环加载与`CommonJS`有本质不同。`ES6`模块是动态引用，如果使用`import`从一个模块加载变量（即`import foo from 'foo'`），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者保证，真正取值的时候，能够取到值。  
            ```js
            // a.mjs
            import { bar } from'./b';
            console.log('a.mjs');
            console.log(bar);
            export let foo = 'foo';
            
            // b.mjs
            import { foo } from'./a';
            console.log('b.mjs');
            console.log(foo);
            export let bar = 'bar';
            ```
            运行结果如下：
            ```
            b.mjs
            ReferenceError: foo is not defined
            ```
            执行过程如下：
            * 执行`a.mjs`以后，引擎发现它加载了`b.mjs`，因此会优先执行`b.mjs`
            * 执行`b.mjs`的时候，已知它从`a.mjs`输入了`foo`接口，这时不会去执行`a.mjs`，而是认为这个接口已经存在了，继续往下执行
            * 执行到第三行`console.log(foo)`的时候，才发现这个接口根本没定义，因此报错
            解决这个问题的方法，就是让`b.mjs`运行的时候，`foo`已经有定义了。这可以通过将`foo`写成函数来解决。
            ```js
            // a.mjs
            import { bar } from'./b';
            console.log('a.mjs');
            console.log(bar());
            function foo() { return 'foo' }
            export { foo };
            
            // b.mjs
            import { foo } from'./a';
            console.log('b.mjs');
            console.log(foo());
            function bar() { return 'bar' }
            export { bar };
            ```
            最后执行结果为：
            ```
            b.mjs
            foo
            a.mjs
            bar
            ```

### 三. 引用
[前端模块化详解(完整版)](https://juejin.im/post/5c17ad756fb9a049ff4e0a62)

[深入浅出 JavaScript 模块化](https://mp.weixin.qq.com/s/Uxtm-z48FnhK13Zbm15Tcw)

[ES6 系列之模块加载方案](https://github.com/mqyqingfeng/Blog/issues/108)