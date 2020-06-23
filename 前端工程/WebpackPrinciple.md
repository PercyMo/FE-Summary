## Webpack原理

### 一. 构建流程详解

![webpack构建流程详解](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Webpack/02.jpg)

#### 1. 基本概念
* **Entry**：入口，webpack执行构建的第一步将从Entry开始，可抽象成输入。
* **Moudle**：模块，在webpack中一切皆模块，一个模块对应着一个文件。webpack会从配置的Entry开始递归找出所有依赖的模块。
* **Chunk**：代码块，一个Chunk又多个模块组合而成，用于代码合并与分割。
* **Loader**：模块转换器，用于把模块原内容按照需求转换成新内容。
* **Plugin**：扩展插件，在webpack构建流程中的特定时机会广播出对应的事件，插件可以监听这些事件的发生，在特定时机做对应的事情。

#### 2. 流程概括
`Webpack`的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

1. **初始化参数**：从配置文件和`Shell`语句中读取和合并参数，得出最终参数；
2. **开始编译**：用上一步得到的参数初始化`Compiler`对象，加载所有配置的插件，执行run方法，开始执行编译；
3. **确定入口**：根据配置中的`entry`找出所有入口文件；
4. **编译模块**：从入口文件出发，调用所有配置的`Loader`对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
5. **完成模块编译**：在经过的第4步使用`Loader`翻译完所有模块后，得到了每个模块被翻译后最终的内容以及它们之间的依赖关系；
6. **输出资源**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的`Chunk`，再把每个`Chunk`转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
7. **输出完成**：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写到文件系统。

在以上过程中，`Webpack`会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后执行特定的逻辑，并且插件可以调用`Webpack`提供的API，改变`Webpack`的运行结果。

#### 3. 流程细节
1. **初始化**：启动构建，读取和合并配置参数，加载`Plugin`，实例化`Compiler`。
2. **编译**：从`Entry`出发，针对每个`Module`串行调用对应的Loader去翻译文件内容，再找到该`Module`依赖的`Module`，递归地进行编译处理。
3. **输出**：对编译后的`Module`组合成`Chunk`，把`Chunk`转换成文件，输出到文件系统。

![webpack编译流程](http://img.vanilla.ink/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/Webpack/01.png?x-oss-process=image/resize,w_250)

1. **初始化阶段**  

    | 事件名 | 解释 |
    |:-|:-|
    | 初始化参数 | 从配置文件和Shell语句中读取与合并参数，得出最终参数。这个过程中还会执行配置文件中的插件实例化语句`new Plugin()`。 |
    | 实例化Compiler | 用上一步得到的参数初始化Compiler实例，Compiler负责文件监听和启动编译。Compiler实例中包含了完整的webpack配置，全局只有一个Compiler实例。 |
    | 加载插件 | 一次调用插件的apply方法，让插件可以监听后续的多所有事件节点。同时给插件传入compiler实例的引用，以方便插件通过compiler调用webpack提供的API |
    | environment | 开始应用Node.js风格的文件系统到compiler对象，以方便后续的文件寻找和读取。 |
    | entry-option | 读取配置的Entrys，为每个Entry实例化一个对应的EntryPlugin，为后面该Entry的递归解析工作做准备。 |
    | after-plugins | 调用完所有的内置的和配置的插件的apply方法。 |
    | after-resolvers | 根据配置初始化完resolver，resolver负责在文件系统中寻找指定路径的文件。 |

2. **编译阶段**  

    | 事件名 | 解释 |
    |:-|:-|
    | run | 启动一次新的编译 |
    | watch-run | 和run类似，区别在于它是在监听模式下启动的编译，在这个事件中可以获取到是哪些文件发生了变化导致重新启动一次新的编译。 |
    | compilie | 该事件是为了告诉插件一次新的编译将要启动，同时会给插件带上complier对象。 |
    | compilation | 当webpack以开发模式运行时，每当检测到文件变化，一次新的Compilation将被创建。一个Compilation对象包含了当前的模块资源、编译生成资源、变化的文件等。Compilation对象也提供了很多事件回调供插件做扩展。 |
    | make | 一个新的Compilation创建完毕，即将从Entry开始读取文件，根据文件类型和配置的Loader对文件进行编译，编译完后再找出该文件的依赖的文件，递归地编译和解析。 |
    | after-compile | 一次Compilation执行完成。 |
    | invalid | 当遇到文件不存在、文件编译错误等异常时会触发该事件，**该事件不会导致webpack退出**。 |

    在编译阶段中，最重要的要数compilation事件了，因为在compilation阶段调用了Loader完成了每个模块的转换操作，在compilation阶段又包含了很多小的事件：
    | 事件名 | 解释 |
    |:-|:-|
    | build-module | 使用对应的Loader去转换一个模块 |
    | normal-module-loader | 在用Loader对一个模块转换完后，使用acorn解析转换后的内容，输出对应的抽象语法树（AST），以方便webpack后面对代码的分析。 |
    | program | 从配置的入口模块开始，分析其AST，当遇到`require`等导入其它模块语句时，便将其加入到依赖的模块列表，同时对新找出的依赖模块递归分析，最终搞清所有模块的依赖关系。 |
    | seal | 所有模块及其依赖的模块都通过Loader转换完成后，根据依赖关系开始生成Chunk。 |

3. **输出阶段**  

    | 事件名 | 解释 |
    |:-|:-|
    | should-emit | 所有需要输出的文件已经生成好，询问插件哪些文件需要输出，哪些不需要。 |
    | emit | 确定好输出哪些文件后，执行文件输出，可以在这里获取和修改输出内容。 |
    | after-emit | 文件输出完毕。 |
    | done | 成功完成一次完整的编译和输出流程。 |
    | failed | 如果在编译和输出流程中遇到异常导致webpack退出时，就会直接跳转到本步骤，插件可以在本事件中获取到具体的错误原因。 |

TODO: 使用import时，webpack对node_modules里的依赖会做什么

### 二. 输出文件分析
待构建代码：
```js
// main.js 入口文件
const show = require('./show');
show('Webpack');

// show.js
function show(content) {
  window.document.getElementById('app').innerText = 'Hello,' + content;
}

// 通过 CommonJS 规范导出 show 函数
module.exports = show;
```

#### 1. 最基本的构建输出
最终输出代码`bundle.js`：
```js
// webpackBootstrap 启动函数
// modules 即为存放所有模块的数组，数组中的每个元素都是一个函数
(function (modules) {
    // 安装过的模块都存放在这里面
    // 作用是把已经加载过的模块缓存在内存中，提升性能
    var installedModules = {};
    
    // 去数组中加载一个模块moduleId 为要加载模块在数组中的 index
    // 作用和 Node.js 中的 require 语句相似
    function __webpack_require__(moduleId) {
        // 如果需要加载的模块已经被加载过，就直接从内存缓存中返回
        if (installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }

        // 如果缓存中不存在要加载的模块，就新建一个模块，并把它存在缓存中
        var module = installedModules[moduleId] = {
            // 模块在数组中的 index
            i: moduleId,
            // 该模块是否已经加载完毕
            l: false,
            // 该模块的导出值
            exports: {}
        };

        // 从 modules 中获取 index 为 moduleId 的模块对应的函数
        // 再调用这个函数，同时把函数需要的参数传入
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
        // 把这个模块标记为已加载
        module.l = true;
        // 返回这个模块的导出值
        return module.exports;
    }

    // Webpack 配置中的 publicPath，用于加载被分割出去的异步代码
    __webpack_require__.p = "";
    
    // 使用 __webpack_require__ 去加载 index 为 0 的模块，并返回该模块导出的内容
    // index 为 0 的模块就是 main.js 对应的文件，也就是执行入口模块
    // __webpack_require__.s 的含义是启动模块对应的 index
    return __webpack_require__(__webpack_require__.s = 0);
})(
    // 所有的模块都存放在了一个数组里，根据每个模块在数组的 index 来区分和定位模块
    [
        /* 0 */
        (function(module, exports, __webpack_require__) {
            // 通过 __webpack_require__ 规范导入 show 函数，show.js 对应的模块 index 为 1
            const show = __webpack_require__(1)
            show('Webpack');
        }),
        /* 1 */
        (function(module, exports) {
            function show(content) {
                window.document.getElementById('app').innerText = 'Hello,' + content;
            }
            // 通过 CommonJS 规范导出 show 函数
            module.exports = show;
        })
    ]
);
```
以上代码其实是一个立即执行函数，可以简写为：
```js
(function (modules) {
    
    // 模拟 require 语句
    function __webpack_require__() {
    }
    
    // 执行存放模块数组中的第0个模块
    __webpack_require__(0)
})([/* 存放所有模块的数组 */])
```
`bundle.js`能直接运行在浏览器中的原因在于输出的文件中通过`__webpack_require__`函数定义了一个可以在浏览器中执行的加载函数俩模拟`Node.js`中的`require`语句。  
原来一个个独立的模块文件被合并到了一个单独的bundle.js的原因在于浏览器不能像Node.js那样快速地去本地加载一个个模块文件，而必须通过网络请求去加载还未得到的文件。如果模块数量很多，加载时间会很长，因此把所有模块都放在了数组中，执行一次网络加载。  
webpack在`__webpack_reqire__`函数的实现中还做了缓存优化：执行加载过的模块会缓存在内存中，当该模块被第二次访问时会直接去内存中读取被缓存的返回值。

#### 2. 分割代码时的输出

### 三. 编写 Loader

### 四. 编写 Plugin

### 五. Webpack 调试

### 六. 实现一个乞丐版 Webpack

### 七. 引用
[webpack原理（《深入浅出 webpack》）](https://webpack.wuhaolin.cn/5%E5%8E%9F%E7%90%86/5-1%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E6%A6%82%E6%8B%AC.html)

[细说 webpack 之流程篇（淘宝FED）](https://fed.taobao.org/blog/2016/09/10/webpack-flow/)
