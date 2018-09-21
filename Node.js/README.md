### 《深入浅出node.js》学习笔记

#### 预备知识

##### 1. npm
```
// 命令对应的相关使用帮助
$ npm <commond> -h
```

```
// 查看对应模块的package.json文件以及npm仓相关属性
$ npm view socket.io
```

##### 2. JavaScript对应基础知识
1. 函数的参数数量

    函数有个很有意思的属性——参数数量，该属性指明函数声明时可接受的参数数量
    ```js
    var a = function(a, b, c) {};
    a.length == 3   // true
    ```
    尽管在浏览器端很少使用，但非常重要，很多流行的Node.js框架就是通过此属性根据不同参数个数提供不同的功能

##### 3. 阻塞与非阻塞IO
1. PHP与Node在基础架构上有明显区别。Node采用一个长期运行的进程，相反，Apache会产出多个线程（每个请求一个线程），每次都会刷新状态。在PHP中，当解释器再次执行时，变量会被重新赋值，而Node则不然。因此，在Node中，需要对回调函数如何修改当前内存中的变量（状态）特别小心。
2. 事件轮询是Node IO的基础核心。许多优秀的Node模块都是非阻塞的，执行任务也都采用了异步的方式。

##### 4. Node中的js
1. 在默认情况下，每个模块都会暴露出一个空对象。如果想在该对象上添加属性，那么使用`exports`即可。
    ```js
    exports.name = 'john';
    exports.data = 'this is some data';

    var privateVariable = 5;

    exports.getPrivate = function() {
        return privateVariable;
    };
    ```
    或者，彻底重写`module.exports`
    ```js
    // module.js
    function Person(name) {
        this.name = 'Tom';
    };

    Person.prototype.talk = function() {
        console.log('name:', this.name);
    };

    module.exports = Person;

    // main.js
    var Person = require('./module_a');

    var tom = new Person();
    tom.talk();
    ```

##### 5. 命令行工具(CLI)以及FS API
1. Node.js 文件系统（fs 模块），是唯一一个同时提供同步和异步API的模块。

    例如读取文件内容的函数有异步的 `fs.readFile()` 和同步的 `fs.readFileSync()`。

    建议大家使用异步方法，比起同步，异步方法性能更高，速度更快，而且没有阻塞。
2. 流

    process全局对象中包含了三个流对象，分别对应对应三个UNIX标准流：
    * ** stdin **   :   标准输入
    * ** srdout **  :   标准输出
    * ** stderr **  :   标准错误
    简而言之，当持续不断地对数据进行读写时，流就出现了。
3. ANSI转义码
    ```js
    console.log('\033[90m' + '这里是一段文本' + '\033[39m');
    ```
    * \033表示转义序列的开始
    * [表示开始颜色设置
    * 90表示前景色为灰亮色
    * m表示颜色设置结束
3. Stream
    `fs.createReadStream`方法允许为一个文件创建一个可读的stream对象   
    例如：要上传一个很大的文件到web服务器，这是你无需读取完所有的文件内容在开始上传，使用stream可以大大提速上传过程。
    ```js
    // 必须等到文件读取完毕，载入到RAM，才能处理
    var stream = fs.readFile('input.txt', function(err, contents) {
        // 对文件进行处理
    });

    // 每次读取可变大小的内容快，并且每次读取后触发回调函数
    var stream = fs.createReadStream('input.txt');
    stream.on('data', function(chunk) {
        // 处理文件部分内容
    });
    stream.on('end', function(chunk) {
        // 文件读取完毕
    });
    ```
4. 监视
    Node允许监视文件或目录是否发生变化。发生变化时，分发一个事件，触发指定的回调函数。
    ```js
    var stream = fs.createReadStream('test.txt');

    var files = fs.readdirSync(process.cwd());
    files.forEach(function(file) {
        if (/\.css/.test(file)) {
            fs.watchFile(process.cwd() + '/' + file, function() {
                console.log('  -  ' + file + 'changed!');
            });
        }
    });
    ```
    除`fs.watchFile`之外，还可以使用`fs.watch`来监视整个目录。

##### 6. TCP


#### 第一章 node简介

##### 1. 单线程
单线程的缺点
* 无法利用多核CPU
* 错误会引起整个应用退出，应用的健壮性值得考验
* 大量计算占用CPU导致无法继续调用异步I/O
像浏览器中JavaScript与UI公用一个线程一样，JavaScript长时间执行会导致UI的渲染和响应被中断。

Node采用了与Web Workers相同的思路来解决单线程中大计算量的问题：`child_process`

#### 第二章 模块机制

##### 1. CommonJS的模块规范
1. 【模块引用】
    `require()`方法，这个方法接受模块标识，以此引入一个模块的API到当前的上下文中
2. 【模块定义】
    上下文中提供了`exports`对象用于导出当前模块的方法或者变量，并且它是唯一导出的出口。在模块中，还存在一个`module`对象，它代表模块本身，而`exports`是`module`的属性。在Node中，一个文件就是一个模块，将方法挂载在`exports`对象上作为属性即可定义导出的方式
3. 【模块标识】
    `require()`方法的参数，必须是符合**小驼峰命名**的字符串，可以没有文件名后缀`.js`。

    每个模块具有独立的空间，它们互不干扰。

模块应用代码示例：
```javascript
// math.js
exports.add = function () {
    var sum = 0,
        i = 0,
        args = arguments,
        l = args.length;
    while (i < l) {
        sum += args[i++];
    }
    return sum;
};

// program.js
var math = require('math');
exports.increment = function (val) {
    return math.add(val, 1);
};
```

##### 2. Node的模块实现

Node模块分为两类：一类是Node提供的模块，核心模块；另一类的用户编写的模块，文件模块

模块引入流程：缓存加载 => 路径分析 => 文件定位 => 编译执行

1. 优先从缓存加载

    Node引入过的模块都会进行缓存，以减少二次引入时的开销。不同于浏览器的静态脚本缓存，浏览器只缓存文件，而Node缓存的是编译和执行之后的对象。因此，二次引入时也就不需要分析路径、文件定位和编译执行的过程，大大提高了再次加载模块时的效率。

    核心模块的缓存检查先于文件模块的缓存检查。
2. 路径分析和文件定位

    1. 模块标识符在Node中主要分为以下几类
        * 核心模块，如http、fs、path等
        * .或..开始的相对路径文件模块
        * 以/开始的绝对路径文件模块
        * 非路径形式的文件模块，如自定义的`connect`模块

    2. 【路径形式的文件模】在加载过程中，Node会逐个尝试模块路径中路径，直到找到目标文件为止。因此，当前文件路径越深，模块查找耗时会越多，这是自定义模块的加载速度是最慢的原因。
    3. 【文件扩展名分析】当标识符中不包含文件扩展名，Node会按照`.js、.json、.node`的次序补足扩展名，依次尝试。

        因为Node是单线程，所以这里会引起性能问题。

        **_解决方案一_**：如果是`.node和.json`文件，在传递给`require()`的标识符中带上扩展名，会加快一点速度。

        **_解决方案二_**：同步配合缓存，可以大幅缓解Node单线程中阻塞式调用的缺陷。
3. 模块编译

