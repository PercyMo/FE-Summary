## Webpack原理

### 一. 构建流程详解

![webpack构建流程详解](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/Webpack/02.jpg)

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

![webpack编译流程](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/Webpack/01.png?x-oss-process=image/resize,w_250)

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
使用异步模块，输出内容将发生变化。
```js
// 异步加载 show.js
import('./show').then((show) => {
  show('Webpack');
})
```
重新构建后，除了入口文件 `bundle.js`，还会多构建输出一个异步加载的文件 `0.bundle.js`，内容如下：
```js
// 0.bundle.js
webpackJsonp(
    // 在其它文件中存放着的模块的 ID （没太理解，个人觉得叫文件id更合适些）
    [0],
    // 本文件所包含的模块
    [
        /* 0 */,
        /* 1 */
        // show.js 所对应的模块
        (function (module, exports) {
            function show(content) {
                window.document.getElementById('app').innerText = 'Hello,' + content;
            }

            module.exports = show;
        })
    ]
);
```
```js
// bundle.js
(function(modules) {
    /**
     * webpackJsonp 用于从异步加载的文件中安装模块
     * 把 webpackJsonp 挂载到全局是为了方便在其他文件中调用。
     *
     * @param chunkIds 异步加载的文件中存放的需要安装的模块对应的 chunkId
     * @param moreModules 异步加载文件中存放的需要安装的模块列表
     * @param excuteModules 在异步加载的文件中存放的需要安装的模块都安装成功后，需要执行的模块对应的 index
     */
    window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, excuteModules) {
        // 把 moreModules 添加到 modules 对象中
        // 把所有 chunkIds 对应的模块都标记成已经加载成功
        var moduleId, chunkId, i = 0, resolves = [], result;
        for (;i < chunkIds.length; i++) {
            chunkId = chunkIds[i];
            if (installedChunks[chunkId]) {
                // installedChunks 每一项在 loading 时的结构为 [resolve, reject, promise]
                resolves.push(installedChunks[chunkId][0]); // [0] 指的就是 resolve
            }
            installedChunks[chunkId] = 0;
        }
        for (moduleId in moreModules) {
            if (Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
                modules[moduleId] = moreModules[moduleId];
            }
        }
        while (resolves.length) {
            resolves.shift()();
        }
    }

    // 缓存已经安装的模块
    var installedModules = {};

    // 存储每个 chunk 的加载状态
    // 键 为 chunk 的 id，值为0表示已经加载成功
    var installedChunks = {
        1: 0 // 这里的1猜测为chunk的上限，如果有3个chunk，这里就变成 3:0 了
    };

    function __webpack_require__(moduleId) {
        // ... 省略和上面一样的内容
    }

    /**
     * 用于加载被分割出去的，需要异步加载的 Chunk 对应的文件
     * @param chunkId 需要异步加载的 Chunk 对应的 ID
     * @returns {Promise}
     */
    __webpack_require__.e = function requireEnsure(chunkId) {
        // 从上面定义的 installedChunks 中获取 chunkId 对应的 Chunk 的加载状态
        var installedChunkData = installedChunks[chunkId];
        // 如果加载状态为0表示 Chunk已经加载成功了，直接返回 resolve Promise
        if (installedChunkData === 0) {
            return new Promise(function(resolve) {
                resolve();
            });
        }
        // installedChunkData 不为空且不为0表示该 Chunk 正在网络加载中
        if (installedChunkData) {
            // 返回存放在 installedChunkData 数组中的 Promise 对象
            return installedChunkData[2];
        }

        // installedChunkData 为空，表示该 Chunk 还没有加载过，去加载该 Chunk 对应的文件
        var promise = new Promise(function(resolve, reject) {
            installedChunkData = installedChunks[chunkId] = [resolve, reject];
        })
        installedChunkData[2] = promise;

        // 通过 DOM 操作，往 HTML head 中插入一个 script 标签去异步加载 Chunk 对应的JS文件
        var head = document.getElementsByTagName('head')[0];
        var script = document.createElement('script');
        script.type = 'script/javascript';
        script.charset = 'utf-8';
        script.async = true;
        script.timeout = 120000;

        // 文件的路径为配置的 publicPath、chunkId 拼接而成
        script.src = __webpack_require__.p + '' + chunkId + '.bundle.js';

        // 设置异步加载的最长超时时间
        var timeout = setTimeout(onScriptComplete, 120000);
        script.onerror = script.onload = onScriptComplete;

        // 在 script 加载和执行完成时回调
        function onScriptComplete() {
            // 防止内存泄漏
            script.onerror = script.onload = null;
            clearTimeout(timeout);

            // 去检查 chunkId 对应的 chunk 是否安装成功，安装成功时才会存在于 installedChunks 中
            var chunk = installedChunks[chunkId];
            if (chunk !== 0) {
                if (chunk) {
                    chunk[1](new Error('Loading chunk' + chunkId + 'failed.'));
                }
                installedChunks[chunkId] = undefined;
            }
        }

        head.appendChild(script);

        return promise;
    }

    // 加载并执行入口模块
    return __webpack_require__(__webpack_require__.s = 0);
})(
    // 存放所有没有经过异步加载的，随着执行入口文件加载的模块
    [
        // main.js 对应的模块
        (function(module, exports, __webpack_require__) {
            // 通过 __webpack_require__.e 去异步加载 show.js 对应的 chunk
            __webpack_require__.e(0).then(__webpack_require__.bind(null, 1)).then((show) => {
                show('Webpack');
            })
        })
    ]
);
```
> 待证实，暂称之“小毛猜想”
> 1. 关于moduleId，webpack会将所有的同步和异步模块一起计数，如果同步模块已经有了2个坑位，那异步模块的id就是从2开始了
> 2. 分割代码的写法略有不同（可能是出于优化考虑），id小于3时，`webpackJsonp([moduleId],[(function() {})])`，反之：`webpackJsonp([moduleId],{3: (function() {})})`

这里的`bundle.js`和上面所讲的`bundle.js`非常相似，区别在于：
* 多了一个`__webpack_require__.e`用于加载被分割出去的，需要异步加载的chunk对应的文件；
* 多了一个`webpackJsonp`函数用于从异步加载的文件中安装模块。

在使用了`CommonsChunkPlugin`去提取公共代码时输出的文件和使用了异步加载输出的文件是一样的，都会有`__webpack_require__.e`和`webpackJsonp`。原因在于提取公共代码和异步加载本质上都是代码分割。

### 三. 编写 Loader
#### 1. loader执行顺序
以处理scss为例，
1. `sass-loader` 把 `scss` 转成 `css`
2. `sass-loader` 输出的 `css` 交给 `css-loader` 处理，找出`css`中依赖的资源、压缩`css`等
3. `css-loader` 输出的 `css` 交给 `style-loader` 处理，转换成通过脚本加载的JS代码

loader的处理过程是有顺序的链式执行，先`sass-loader`，再`css-loader`，最后`style-loader`。`use`数组中，从右至左执行。
```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.scss$/,
                use: [
                    'style-loader',
                    {
                        loader: 'css-loader',
                        // css-loader 配置
                        options: {
                            minimize: true
                        }
                    },
                    'sass-loader'
                ]
            }
        ]
    }
}
```

#### 2. loader的职责
一个loader的职责是单一的，只需要完成一种转换。如果一个源文件需要经历多步转换才能正常使用，就通过多个loader去转换。在调用多个loader去转换一个文件时，每个loader会链式地顺序执行，第一个loader将会拿到需处理的原内容，上一个loader处理后的结果会传给下一个接着处理，最后的loader将处理后的结果返回给webpack。

#### 3. loader编写规则
1. 一个最简单的loader
    ```js
    module.exports = function (source) {
        // source 为 compiler 传递给 loader 的一个文件的原内容
        // 该函数直接把原内容返回，相当于loader没有做任何转换
        return source;
    };
    ```

2. loader运行在Node.js中，你可以调用任何Node.js自带的API，或者安装第三方模块进行调用。
    ```js
    const sass = require('node-sass');
    module.exports = function (source) {
        return sass(source);
    };
    ```

3. 获得loader的options
    ```js
    const loaderUtils = require('loader-utils');
    module.exports = function (source) {
        // 获取到用户给 sass-lader 传递的 options 参数
        const options = loaderUtils.getOptions(this);
        return source;
    };
    ```

4. 返回其它结果  
    以用`babel-loader`转换ES6代码为例，它还需要输出转换后的ES5代码对应的`Source Map`，以方便调试源码。为了把`Source Map`也一起随着ES5代码返回给webpack，可以这样写：
    ```js
    module.exports = function (source) {
        // 通过 this.callback 告诉 webpack 返回结果
        this.callback(null, source, sourceMaps);
        // 当使用 this.callback 返回内容时，该 loader 必须返回 undefined，
        // 以让 webpack 知道该 loader 返回的结果在 this.callback 中，而不是 return 中
        return;
    };
    ```
    `this.callback`是webpack给loader注入的API，以方便webpack和loader之间通讯。`this.callback`的详细使用方法如下：
    ```js
    this.callback({
        // 当无法转换原内容时，给 webpack 返回一个 Error
        err: Error | null,
        // 原内容转换后的内容
        content: String | Buffer,
        // 用于把转换后的内容得出原内容的 Source Map，方便调试
        sourceMap?: SourceMap,
        // 如果本次转换的原生内容生成了 AST 语法树，可以把这个 AST 返回，
        // 以方便之后需要 AST 的 loader 复用该 AST，避免重复生成 AST，提升性能
        abstractSyntaxTree?: AST
    });
    ```
    > Source Map的生成很耗时，通常在开发环境下才生成Source Map，其他环境下不用生成，以加速构建。为此webpack 为 laoder 提供了 `this.sourceMap` API 告诉 loader 当前构建环境下用户是否需要 Source Map。

5. 同步和异步  
    当转换步骤存在异步时：
    ```js
    module.exports = function(source) {
        // 告诉 webpack 本次转换是异步的，loader 会在 callback 中回调结果
        var callback = this.async();
        someAsyncOperation(source, function(err, result, sourceMaps, ast) {
            // 通过 callback 返回异步执行的结果
            callback(err, result, sourceMaps, ast);
        });
    };
    ```

6. 处理二进制数据  
    webpack默认传给 loader 的原内容都是 UTF-8 格式编码的字符串。但某些场景下，如：file-loader，需要 webpack 给loader传入二进制格式数据。
    ```js
    module.exports = function(source) {
        // 在 exports.row === true时，webpack 传给 loader 的 source 是 Buffer 类型的
        source instanceof Buffer === true;
        // loader 返回值也可以是 Buffer
        // exports.row !== true，loader 也可以返回 Buffer 类型
        return source;
    };
    // 通过 exports.row 属性告诉 webpack 该 loader 是否需要二进制数据，没有该行声明，loader只能拿到字符串
    module.exports.row = true;
    ```

7. 缓存加速  
    有些转换操作需要大量计算非常耗时，如果每次构建都重新执行重复的转换操作，构建将会变得非常缓慢。webpack会默认缓存所有 loader 的处理结果，在需要被处理的文件或者其依赖的文件没有发生变化时，是不会重新调用对应的loader去执行转换操作的。  
    如果想让 webpack 不缓存该 loader 的处理结果：
    ```js
    module.exports = function(source) {
        // 关闭该 loader 的缓存功能
        this.cacheable(false);
        return source;
    };
    ```

#### 4. 其它loader API
[webpack官网 其他loader的API](https://webpack.js.org/api/loaders/)

#### 5. 加载本地 loader
1. **npm link**  
    通过软链将 loader 模块链接到项目的 node_modules 目录下

2. **ResolveLoader**  
    ```js
    // webpack.config.js
    module.exports = {
        resolveLoader: {
            // 去哪些目录下寻找 loader，有先后顺序之分
            modules: ['node_modules', './loaders/']
        }
    };
    ```
    加上配置后，webpack会先去node_modules项目下寻找loader，如果找不到，会再去`./loader/`目录下寻找。

### 四. 编写 Plugin
#### 1. 最简单的插件编写
```js
// BasicPlugin.js
class BasicPlugin {
    // 在构造函数中获取用户给该插件传入的配置
    constructor(options) {
    }

    // webpack 会调用 BasicPlugin 实例的 apply 方法给插件实例传入 Compiler 对象
    apply(compiler) {
        compiler.plugin('emit', function (compilation, callback) {
            // console.log('test', compilation)
        })
    }
}

// 导出 plugin
module.exports = BasicPlugin;
```
使用该plugin，`webpack.config.js`中的配置
```js
const BasicPlugin = require('./BasicPlugin.js');
module.exports = {
    plugins: [
        new BasicPlugin(options)
    ]
};
```
webpack启动后，在读取配置的过程中会先执行`new BasicPlugin(options)`初始化`BasicPlugin`获得其实例。在初始化compiler对象后，再调用`BasicPlugin.apply(compiler)`给插件实例传入`compiler`对象。插件实例在获取到`compiler`对象后，就可以通过`compiler.plugin(事件名称, 回调函数)`监听到webpack广播出来的事件，并且可以通过compiler对象去操作webpack。

#### 2. Compiler和Compilation
* compiler对象包含了webpack环境所有的配置信息，包含options、loaders、plugins，这个对象在webpack启动时被实例化，它是全局唯一的，可以简单理解为webpack实例。
* compilation对象包含了当前的模块资源、编译生成资源、变化的文件等。当webpack以开发模式运行时，每当检测到一个文件变化，一次新的compilation将被创建。

compiler和compilation区别在于：compiler代表了整个webpack从启动到关闭的声明周期，而compilation只是代表了一次新的编译。

#### 3. 事件流
webpack通过[Tapable](https://github.com/webpack/tapable)来组织事件流。webpack会在运行过程中广播事件，插件只需要监听它所关心的事件，就能加入到这条生产线，改变生产线的运作。webpack的事件流机制保证了插件的有序性。  
webpack的事件流机制应用了观察者模式，和Node.js中的EventEmitter非常相似。Compiler和Compilation都继承自Tapable，可以直接在Compiler和Compilation对象上广播和监听事件。
```js
/**
* 广播出事件
* event-name 为事件名称，注意不要和现有的事件重名
* params 为附带的参数
*/
compiler.apply('event-name', params);

compiler.plugin('event-name', function(params) {
});
```
> TODO: 对于同一个事件，例如'emit'，如果在多个插件上监听，只有第一个实例化的插件会被执行。待证实

开发插件注意点：
* 只要能拿到Compiler或Compilation对象，就能广播出新的事件，所以在新开发的插件中也能广播出事件，给其他插件监听使用。
    > TODO: 这里的坑感觉有点深了，关于Tapable，和Hooks单独写一篇
    > [编写自定义webpack插件从理解Tapable开始](https://juejin.im/post/5dcba29f6fb9a04abb01fd77)
    > [Webpack揭秘——走向高阶前端的必经之路](https://juejin.im/post/5badd0c5e51d450e4437f07a#heading-18)
    > [揭秘webpack插件工作流程和原理](https://mp.weixin.qq.com/s/svhh8BJdxXGJuCuumJiBgA)
* 传给每个插件的Compiler或Compilation对象都是同一个引用。也就是说在一个插件中修改了Compiler或Compilation对象上的属性，会影响到后面的插件。
* 有些事件是异步的，这些异步的事件会附带两个参数，第二个参数为回调函数，在插件处理完任务时需要调用回调函数通知webpack，才会进入下一处理流程：
    ```js
    compiler.plugin('emit', function(compilation, callback) {
        // 处理完成后执行callback通知webpack
        // 如果不执行callback，运行流程将会一直卡在这里不往下执行
        callback();
    });
    ```

#### 4. 常用API
1. **读取输出资源、代码块、模块及其依赖**  
    在`emit`事件发生时，代表源文件的转换和组装已经完成，在这里可以读取到最终将输出的资源、代码块、模块及依赖，并可以修改输出自愿的内容。
    ```js
    class Plugin {
        apply(compiler) {
            compiler.plugin('emit', function (compilation, callback) {
                // compilation.chunks 存放所有代码块，是一个数组
                compilation.chunks.forEach(function (chunk) {
                    // chunk 代表一个代码块
                    // 代码块由多个模块组成，通过 chunk.forEachModule 能读取组成代码块的每个模块
                    chunk.forEachModule(function (module) {
                        // module 代表一个模块
                        // module.fileDependencies 存放当前模块的所有依赖的文件路径，是一个数组
                        module.fileDependencies.forEach(function (filePath) {
                        });
                    });

                    // webpack 会根据 chunk 去生成输出的文件资源，每个 chunk 都对应一个及以上的输出文件
                    // 例如在 chunk 中包含了 css 模块，并且使用了 ExtractTextPlugin 时，
                    // 该 chunk 就会生成 .js 和 .css 两个文件
                    chunk.files.forEach(function (filename) {
                        // compilation.assets 存放当前所有即将输出的资源
                        // 调用一个输出资源的 source() 能获取到输出资源的内容
                        const source = compilation.assets[filename].source();
                    });
                });

                // 这是一个异步事件，需要调用 callback 通知 webpack 本次事件监听处理结束
                callback();
            })
        }
    }
    ```    

2. **监听文件变化**  
    当入口模块或其依赖的模块发生变化时，webpack就会触发一次新的compilation。
    ```js
    // 当依赖的文件发生变化时会触发 watch-run 事件
    compiler.plugin('wtach-run', (watching, callback) => {
        // 获取发生变化的文件列表
        const changeFiles = watching.compiler.watchFileSystem.watcher.mtimes;
        // changeFiles 格式为键值对，键为发生变化的文件路径
        if (changeFiles[filePath] !== undefined) {
            // filePath 对饮的文件发生了变化
        }
        callback();
    });
    ```
    默认情况下，webpack只会监听入口和其依赖的模块是否发生变化，在有些情况下项目可能需要引入新文件，例如引入一个html。由于js不会去导入html文件，因此编辑html时不会触发新的compilation。为了监听html的变化，需要把html加入到依赖列表中：
    ```js
    compiler.plugin('after-compile', (compilation, callback) => {
        compilation.fileDependencies.push(path.resolve(__dirname, '../index.html'));
        callback();
    });
    ```

3. **修改输出资源**  
    `emit`事件时所有模块的转换和代码对应的文件已经生成好了，需要输出的资源即将输出，因此`emit`事件是修改、增加、删除webpack输出资源的最后时机。  
    所有需要输出的资源会存放在`compilation.assets`中，`compilation.assets`是一个键值对，键为需要输出的文件名称，值为文件对应的内容。  
    设置`compilation.assets`的代码如下：
    ```js
    compiler.plugin('emit', (compilation, callback) => {
        // 设置名为 fileName 的输出资源
        compilation.assets[filename] = {
            // 返回的文件内容
            source: () => {
                // fileContent 既可以是代表文本文件的字符串，也可以是二进制Buffer
                return fileContent;
            },
            // 返回文件大小
            size: () => {
                return Buffer.byteLength(fileContent, 'utf8');
            }
        };
        callback();
    });
    ```
    读取`compilation.assets`代码如下：
    ```js
    compiler.plugin('emit', (compilation, callback) => {
        const asset = compilation.assets[fileName];
        asset.source();
        asset.size();
        callback();
    });
    ```

4. **判断webpack使用了哪些插件**  
    开发一个插件时可能需要根据当前配置是否使用了其他某个插件而做下一步决定，因此需要读取webpack当前的插件配置情况。
    ```js
    function hasExtractTextPlugin(compiler) {
        const plugins = compiler.options.plugins;
        return plugins.find(plugin => plugin.__proto__.constructor === ExtractTextPlugin) !== null;
    }
    // 没懂constructor === ExtractTextPlugin 是怎么判断的，是否是伪代码
    // constructor.name === 'ExtractTextPlugin'可行
    ```

### 五. Webpack 调试
由于webpack运行在Node.js之上，调试webpack就相当于调试Node.js程序。  
VS Code断点调试，`--inspect`，单独解释下。  

### 六. 其他
> 一点小小提醒的给自己，配置webpack时，一定注意webpack自身版本和所用的插件、loader版本是否匹配，遇到过不止一次这样的问题，webpack给出的提示莫名其妙，花费了许多时间都解决不了的报错，也许只是版本问题，换个版本就好了。
> 遇到疑难杂症的时候，一定想一下是否可能是版本的问题导致。
> ***千万谨记！千万谨记！千万谨记！***

### 七. 引用
[webpack原理（《深入浅出 webpack》）](https://webpack.wuhaolin.cn/5%E5%8E%9F%E7%90%86/5-1%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E6%A6%82%E6%8B%AC.html)

[细说 webpack 之流程篇（淘宝FED）](https://fed.taobao.org/blog/2016/09/10/webpack-flow/)
