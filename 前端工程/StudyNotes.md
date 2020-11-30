## 《前端工程化-体系设计与实践》笔记
### 一. 前端工程简史
#### 1. 目前的前后端协作流程及工程化程度
![前后端协作流程](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/01_v1.png?x-oss-process=image/resize,w_500)

1. 开发层面
    * ES规范与浏览器兼容性不一致
    * CSS的弱编程能力
    * 资源定位
    * 图片压缩/base64内嵌/CSS Sprites
    * 模块依赖分析和压缩打包
2. 协作层面
    * JavaScript部分逻辑依赖接口API
3. 部署层面
    * 部分资源需要借助后端工程师部署

#### 2. 工程化方案架构
目前前端工具大体分为3类：
1. 工作流管理工具，如Grunt、Gulp
2. 构建工具，如webpack、rollup
3. 整体解决方案，如FIS、WeFlow
最终的选择是webpack。  
TODO: 这里可以专门对比下这些工具的差异

#### 3.工程化方案的整体架构
![工程化方案的整体架构](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/02.png?x-oss-process=image/resize,w_400)

* 暴露给用户层的有两种接口：命令行调用各功能模块的接口和配置接口
* 平台层分为4个功能模块：脚手架、本地服务器、构建以及部署模块
* 内核层是各个功能模块的内核，脚手架使用Yeoman，本地服务由Node.js的Express框架承载，构建功能模块围绕webpack打造，部署基于SSH协议实现。
* 以上所有的功能实现均是建立在Node.js平台上的

### 二. 脚手架
#### 1. 脚手架的功能和本质
脚手架的功能是创建项目初始文件，本质是方案的封装。

#### 2. 脚手架在前端工程中的角色和特征
1. 脚手架工具要解决的最切实的问题：
    * 快速生成配置
    * 降低框架学习成本
    * 令业务开发人员关注业务逻辑本身
2. vue-cli就是解决这些问题的一个非常典型的例子。创建Vue项目的同时，vue-cli提供了模板选择、编译以及本地开发服务器等功能模块。对于使用vue-cli创建的vue项目，业务开发人员无需操心复杂的webpack配置。只需关注业务逻辑开发本身，这很大程度上降低了开发的时间成本。
3. 执行环境分为4类：本地环境、集成平台环境、测试环境以及生产环境
    * 本地环境是指开发人员的本机环境
    * 集成平台环境指的是云管理平台或者持续集成平台环境
    * 测试环境指的是集成测试阶段测试工程师对产品进行仿真模拟测试的特定沙箱环境
    * 生产环境指的是产品交付给用户的真实环境

    ![工作流](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/03.png?x-oss-process=image/resize,w_500)

    不论前端工程化是最简单的本地工具链，还是集大成者的持续集成阶段，脚手架的执行环境始终局限于本地。这给脚手架工具带来了一个必须解决的问题：操作系统兼容性。

    小贴士：有的团队没有本机环境，而是提供虚拟机供开发人员使用。可以规避由操作系统差异引起的额外开发成本，工程化工具也不必考虑操作系统兼容性问题。

4. 多样性的实现模式
    * 脚手架与构建模块如何协作  

    ![脚手架协作](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/04.png?x-oss-process=image/resize,w_500)
#### 3. 脚手架功能规划
1. 命令行工具  
    | 命令 | 功能 |
    |:- |:- |
    | boi new | 脚手架 |
    | boi build | 构建 |
    | boi server | 开发本地服务器 |
    | boi deploy | 远程部署 |
2. 构建功能规划  
3. 环境区分  
    | 名称 | 运行环境 | 执行时机 |
    |:- |:- |:- |
    | dev | 开发 | 本地服务器 |
    | testing | 测试 | 构建和部署 |
    | production | 生产 | 构建和部署 |

4 集成Yeoman封装脚手架方案  
    TODO: Yeoman的使用可单独整理下，实践手写一个脚手架

### 三. 构建
#### 1. 构建能解决的问题
构建需要解决的问题可归纳为3类：面向语言、面向优化、面向部署
1. 将源码 转为宿主浏览器可执行的代码
    * 领先于浏览器实现的ESMAScript规范编写的JS代码
    * LESS/SASS预编译语法编写的CSS代码
    * Jade/EJS等模板语法编写的HTML代码

    编译后，分别对应：

    * ECMAScript规范的转译
    * CSS预编译语法转译
    * HTML模板渲染

2. 依赖打包——分析文件依赖关系，将同步依赖的文件打包在一起，减少HTTP请求数量
3. 资源嵌入——比如小于10KB的图片编译为base64格式嵌入文档，减少一次HTTP请求
4. 文件压缩——减少文件体积，缩短请求时间
5. hash指纹——通过给文件名加入hash指纹，以应对浏览器缓存策略
6. 域名/路径改变——开发环境与线上环境的域名肯定是不同的，不同类型的资源甚至部署于不同的CDN服务器上
7. 文件名改变——经过构建之后文件名被加上hash指纹，内容改动导致hash指纹的改变

#### 2. 规范和API设计原则
1. 配置API设计  
    ```
    ├── dest                    # 编译输出目录
    ├── package.json            
    └── src                     # 源文件目录
        ├── assets              # 媒体资源
        ├── index.html          # html入口源文件
        ├── js                  # webpack编译入口js源文件
        │   ├── main.auth.js
        │   └── main.home.js
        └── style               # style源文件
            ├── main.auth.scss
            └── main.home.scss
    ```
    如此简单的项目，使用webpack进行构建，基础配置如下：
    ```js
    // 基于 webpack 2.x
    const config = {
        // 编译入口文件
        entry: {
            home: './src/js/main.home.js',
            auth: './src/js/main.auth.js'
        },
        // 编译输出目录和文件名
        output: {
            path: './dest',
            filename: '[name].[thunkhash:8].js',
            publicPath: '/'
        },
        // 分发编译流程
        module: {
            rules: [{
                test: /\.js$/,
                use: ['babel-loader'],
            }, {
                test: /\.scss$/,
                use: ExtractTextPlugin.extract({
                    use: [{
                        loader: 'css-loader',
                        options: {
                            url: true,
                            minimize: true,
                            importLoaders: 1
                        }
                    }, 
                    'sass-loader'
                    ],
                    fallback: 'style-loader',
                    publicPath: '/'
                })
            }]
        },
        // 插件
        plugins: [
            // Uglify插件
            new webpack.optimize.UglifyJsPlugin({
                compress: {
                    warnings: false
                },
                sourceMap: false
            }),
            // css 导出插件
            new ExtractTextPlugin({
                filename: './dest/css/[name].[contenthash:8].css'
            }),
            // 编译html插件
            new HtmlWebpackPlugin({
                filename: 'index.html',
                template: 'index.html',
                inject: true,
                minify: {
                    removeComments: true,
                    collapseWhitespace: true,
                    removeAttributeQuotes: true
                }
            })
        ]
    }
    ```
    webpack毋庸置疑是一款优秀的构建工具，但拥有强大功能的同时，也具备高度的配置复杂度，开发者需要花费大量时间阅读文档才能够了解各个配置项对应的具体功能。  

    经过Boi封装后针对上述需求的配置方案如下：
    ```js
    boi.spec('basic', {
        // 源码目录
        source: 'src',
        // 编译输出目录
        output: 'dest'
    });
    boi.spec('js' , {
        // JS文件后缀类型
        ext: 'js',
        // JS文件目录，相对于basic.source
        source: 'js',
        // JS文件输出目录，相对于basic.output
        output: 'js',
        // JS入口文件的前缀，入口文件的命名规则为[mainFilePrefix].*.[ext]
        mainFilePrefix: 'main',
        // 是否压缩混淆
        uglify: true
    });
    boi.spec('style', {
        // style文件后缀类型
        ext: 'scss',
        // style文件目录，相对于basic.source
        source: 'style',
        // style文件输出目录，相对于basic.output
        output: 'style'
    });
    ```
    与直接配置webpack相比，不仅降低了配置自身的复杂度，而且各配置项对应的功能一目了然，一线业务开发人员能够快速上手。  
    当然，上层封装的缺点是自身封装的构建方案比较局限，不能将webpack全部配置开放给用户，所以需要制定扩展策略以便于用户解决特殊需求，也就是插件机制。

2. 编程范式约束  
    工程化方案作为一种服务，应该尽量降低对项目产生的负面影响。  
    
    ```css
    // FIS会对CSS源代码中路径带?__sprite标记的图片进行合并。
    .icon__home {
        background-image: url('home.png?__sprite');
    }

    .icon__auth {
        background-image: url('auth.png?__sprite');
    }
    ```
    经过构建之后产出的CSS代码为：
    ```css
    .icon__home,.icon__auth {
        background-image: url('app_x.png');
    }
    .icon__home {
        background-position: 0px 0px;
    }
    .icon__auth {
        background-position: 0px -20px;
    }
    ```
    上面的例子说明了对源代码的捆绑如何限制了可移植性。

#### 3. 使用ES最新规范开发的优势及对应的构建方案设计
1. Babel——真正意义的JS编译  
    Babel的作用简单概括就是将浏览器未实现的ES规范语法转化为可运行的低版本语法。  
    在Babel出现之前，比如我们必须在兼容IE8浏览器与`bind()`函数之间进行抉择，结果要么是放弃`bind()`，要么自行封装或者使用jQuery等三方类库实现相似的效果。那个时代的JS编程是直接面向目标浏览器的，技术选型、API使用最重要的考虑因素是浏览器兼容性。甚至项目的功能有时也需要做一定的折中以便能够在低版本浏览器上运行。  
    ES6的`class`经Babel转化为ES5的`prototype`实现：
    ```js
    class Test {
        constructor() {
            this.name = 'test'
        }
        getName() {
            return this.name
        }
    }
    ```
    经Babel转化后输出的代码如下：
    ```js
    "use strict";

    function _classCallCheck(instance, Constructor) {
        if (!(instance instanceof Constructor)) {
            throw new TypeError("Cannot call a class as a function");
        }
    }

    function _defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }

    function _createClass(Constructor, protoProps, staticProps) {
        if (protoProps) _defineProperties(Constructor.prototype, protoProps);
        if (staticProps) _defineProperties(Constructor, staticProps);
        return Constructor;
    }

    var Test = /*#__PURE__*/function () {
        function Test() {
            _classCallCheck(this, Test);

            this.name = 'test';
        }

        _createClass(Test, [{
            key: "getName",
            value: function getName() {
                return this.name;
            }
        }]);

        return Test;
    }();
    ```

    Babel将class转化为两部分：工厂函数代码和创建class本身的代码。这种模式的作用会随着项目代码量的增大而被逐渐放大。  
    Babel根据具体配置参数决定编译输出的具体语法，所有的配置参数均由项目针对的浏览器特性决定，比如兼容IE8需要配置Babel将部分ES6语法转化为ES3语法。

2. 结合webpack与babel实现JS构建  
    一个典型的`babel-loader`配置项：
    ```js
    // 基于 webpack 4.x | babel-loader 8.x | babel 7.x
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /(node_modules|bower_components)/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            ['@babel/preset-env', {
                                'targets': {
                                    'browsers': ['last 2 versions', 'ie >= 8']
                                }
                            }]
                        ],
                        plugins: ['@babel/plugin-proposal-object-rest-spread']
                    }
                }
            }
        ]
    }
    ```

    1. babel-preset-env  
        Babel提供了丰富的preset插件供开发者根据项目具体需求进行自由搭配。例如babel-preset-es2015插件是一个集合，包含了将ES6转化为ES5对应语法的所有插件。  

        ![Babel插件](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/05.jpg?x-oss-process=image/resize,w_500)

        但在实际开发过程中，根据项目针对浏览器版本的不同，部分ES6语法并不需要转化为ES5。比如项目只需要兼容Chrome59以上的版本，浏览器自身已经实现了对箭头函数的支持，源码中的箭头函数就没必要转成ES5语法。但要想自己搭配各个独立的插件以便剔除babel-preset-es2015中的冗余部分，并不是件容易的事，babel-preset-env节省了你搭配插件的时间。  
        ```js
        presets: [
            ['@babel/preset-env', {
                'targets': {
                    'chrome': 59
                }
            }]
        ]
        ```
    2. babelrc文件  
        使用babel-loader在webpack中进行options配置有一个致命缺陷：编译只会匹配符合test规则的文件。`/.js$/`只会处理后缀名为.js的文件。但`<script>`标签内的JS代码或者.vue单文件组件就不会被处理。  
        解决这个问题的办法就是使用.babelrc文件取代webpack配置中babel-loader的options。  

#### 4. CSS预编译器和PostCSS各自的作用和区别及对应的构建方案设计
1. CSS的缺陷  
    1. **浏览器实现不理想甚至实现方案各一**。各种css前缀，hack
    2. **CSS的弱编程能力**。不支持嵌套、运算、变量、复用

2. CSS预编译器  
    SASS、LESS、Stylus，从以下几个方面提升CSS的开发效率
    1. 增强编程能力
    2. 增强源码可复用性
    3. 增强源码可维护性
    4. 更便于解决浏览器兼容性
    
    不同的预编译器特性虽然有所差异，但核心功能均围绕这些目标打造：
    * 嵌套：所有预编译器都支持
    * 变量：增强可编程能力
    * mixin/继承：解决hack和代码复用问题
    * 运算：增强可编程能力
    * 模块化：利于代码复用，提高可维护性

3. PostCSS  
    CSS预编译器的语法是各成一派的，而PostCSS鼓励开发者使用规范的CSS原生语法编写源代码，然后配置编译器需要兼容的浏览器版本，最后经过编译将源码转化为目标浏览器可用的CSS代码。从理念上更加接近Babel。  
    但是，PostCSS并不能完全弥补CSS的缺陷，所以目前普遍的方案是将CSS预编译与PostCSS综合在一起。  

    ![CSS预编译+PostCSS](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/06.png?x-oss-process=image/resize,w_400)

    * 使用CSS预编译弥补CSS源码的弱编程能力，比如变量、运算、继承、模块化等
    * 使用PostCSS处理针对浏览器的需求，比如autoprefix、自动CSS Sprites等

4. 结合webpack实现CSS构建  
    ```js
    {
        test: /\.(sa|sc|c)ss$/,
        use: [
            // MiniCssExtractPlugin.loader,
            'style-loader',
            'css-loader',
            'postcss-loader',
            'sass-loader',
        ]
    }
    ```
    .sass后缀的文件依次经过sass-loader、css-loader和style-loader编译。  
    css-loader的作用是解析css源文件并获取其引用的资源，比如@import引用的模块、url()引用的图片等，然后根据webpack配置编译这些资源。style-loader负责将CSS代码通过`<style>`标签插入HTML文档中，所以如果独立导出css文件（`MiniCssExtractPlugin`）就不再需要style-loader。css-loader必须在style-loader之前执行。

#### 5. JS模块化规范与webpack支持性
CommonJS/AMD/CMD/ES6 Module

1. 模块化与组件化  
    严格来说，组件和模块是两个不同的概念。两者的区别主要体现在颗粒度层面。  
    实际上，两者边界模糊不清，没必要为这种概念争执。  
    所谓的模块化，粗俗的讲，就是把一大坨代码，一铲一铲分成一小坨一小坨的代码。  

2. 模块化开发的价值  
    1. 可维护性
    2. 减少全局变量污染
    3. 可复用性
    4. 方便管理依赖关系：在模块化规范没有完全确定的时候，模块之间相互依赖的关系非常模糊，完全取决于js文件引用的顺序。不仅依赖模糊，且难以维护。

3. 异步模块构建差异  
    CommonJS/AMD/ES6 Module 3种规范的差异除在编写代码时存在的差异以外，在使用webpack对异步模块进行构建时也存在命名上的差异。  
    webpack的output选项：
    ```js
    // chunkFilename 是异步文件的名称规则
    output: {
        path: './dest,
        filename: '[name].[chunkhash].js',
        chunkFilename: '[name].[chunkhash:8].js'
    }
    ```
    AMD本身具备异步加载的功能
    ```js
    // main.home.js
    setTimeout(async function() {
        await require(['./a.js'])
        console.log('load module a')
    }, 2000)
    ```
    经webpack构建后产出以下文件：  

    ![webpack构建AMD异步模块](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/07.png?x-oss-process=image/resize,w_500)

    构建输出的文件名为`1.087f61de.js`，而且其`Chunk Names`为空值。实际上，文件名中的1是模块的id。这种命名方式是没有语义的，对于线上错误跟踪也不方便。可见webpack对于AMD这种已经落后于时代的规范支持不佳。  

    CommonJS和ES6 Module规范并没有支持异步加载的功能。webpack通过require.ensure API和import()函数实现异步加载。
    ```js
    // main.home.js
    // webpack提供了特殊注释来定义异步模块的 Chunk Name
    setTimeout(async function() {
        await import(/* webpackChunkName: "mao" */ './a.js')
        console.log('load module a')
    }, 2000)
    ```
    经webpack构建后产出如下文件：  

    ![webpack构建ES6异步模块](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/08.png?x-oss-process=image/resize,w_500)

#### 6. 针对浏览器缓存策略的构建功能的设计
1. HTTP缓存策略  
    浏览器对静态资源的缓存本质上是HTTP协议的缓存策略，即：强缓存与协商缓存。  
    1. Expries 和 Cache-control
        ```sh
        # HTTP 1.0
        Expires: Thu, 10 Nov 2017 08:45:11 GMT

        # HTTP 1.1
        Cache-control: max-age=2592000
        ```

    2. Last-Modified 和 Etag
        ```sh
        # HTTP 1.0
        Last-Modified: Mon, 10 Nov 2018 09:10:11 GMT

        # HTTP 1.1
        Etag: FpCaYjU-Tbu-QqsYyqFRaibzH_RE
        ```
    
    ![HTTP缓存策略](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/09.png?x-oss-process=image/resize,w_500)

    设置`Cache-control: no-chche`，便可以强制浏览器使用协商缓存。

2. 覆盖更新与增量更新  
    覆盖更新与增量更新都是建立在启用强缓存策略的前提下。  
    hash指纹：通过既定的数据摘要算法（目前使用较广泛的是md5）计算出文件的hash值。  
    1. 覆盖更新  
        ```html
        <head>
            <link rel="stylesheet" href="main.home.css?v=858d5483">
        </head>
        <body>
            <script type="text/javascript" src="main.home.js?v=bbcdaf73"></script>
        </body>
        ```
        将hash指纹作为url参数实现覆盖更新的方案有两个致命缺陷：
        1. 必须保证html文件与改动的静态文件同步更新，否则会出现资源不同步的情况。（这就是为什么以前很多团队总是选择在半夜或凌晨这种网站访问量较小的时间段发布新版本的原因之一。）
        2. 不利于版本回滚。

        由于覆盖更新的缺陷较多且没有较好的解决方案，目前已逐渐被淘汰。
    
    2. 增量更新  
        ```html
        <head>
            <link rel="stylesheet" href="main.home.858d5483.css">
        </head>
        <body>
            <script type="text/javascript" src="main.home.bbcdaf73.js"></script>
        </body>
        ```
        在静态资源使用增量更新的前提下，可以将静态资源先于动态HTML部署，此时静态资源没有引用入口，不会对线上环境产生影响；动态HTML部署后即可在第一时间访问已存在的最新静态资源。这样便解决了覆盖更新部署同步性的问题。  
        另外，增量更新修改了资源文件名，不会覆盖已存在的旧版本文件，运维人员回滚操作时只需回滚HTML即可。这样不仅优化了版本控制，而且还可以支持多版本共存的需求。

3. webpack实现增量更新构建方案  
    1. hash 与 chunkhash
        ```js
        output: {
            // filename: '[name].[hash:8].js'
            filename: '[name].[chunkhash:8].js'
        }
        ```
        hash基于webpack的compilation计算，而compilation对象代表某个版本的资源对应的编译进程，它不是针对单个文件的，而是针对项目中所有参与构建的文件的。所以，如果使用hash作为构建输出文件的指纹，任何一个文件的改动都会影响所有资源的缓存。hash不适用于增量更新的构建场景。  
        chunkhash是一个个“块”依据自身的代码内容计算所得的hash值，所以作为文件名hash指纹的构建结果遵循合理的输出原则。
    
    2. contenthash  
        在webpack中，css源文件必须在JS中引入。webpack默认将构建后的CSS代码合并到引用它的js文件中，此js文件运行时在HTML文档中动态添加`<style>`标签，所以js主文件与css文件的hash指纹（thunkhash）完全一致。  

        ![webpack构建css](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/10.png?x-oss-process=image/resize,w_500)

        contenthash并不是webpack自身的另外一种hash值，而是由MiniCssExtractPlugin插件提供的。可以解耦js与css文件的hash指纹。

### 四. 本地开发服务器
#### 1. 动态构建
假如没有动态构建，工程师将不得不重复着调试 -> 修改源码 -> 手动构建 -> 调试 的工作。  
前端开发过程中需要非常频繁的调试，尤其是CSS效果的实现，每次修改源码后手动执行构建势必会拖慢工作效率。

1. webpack-dev-middleware  
    webpack-dev-server是官方提供的用于搭建本地开发环境的小型HTTP服务框架，提供动态编译、HMR（热更新）等功能。devServer其实是基于webpack-dev-middleware和Express实现的，而webpack-dev-middleware其实是Express的一个中间件。  
    如果你的项目不需要Mock服务，webpack-dev-server完全可以满足需求。
    ```js
    // 实现DevServer基本功能的代码大致如下
    const express = require('express')
    const webpack = require('webpack')
    const webpackMiddleware = require('webpack-dev-middleware')

    // 从 webpack.config.js 文件中读取 webpack 配置
    const config = require('./webpack.config.js')
    // 实例化一个Express app
    const app = express()

    // 用读取到的 webpack 配置实例化一个 Compiler
    const compiler = webpack(config)
    // 给 app 注册 webpackMiddleware 中间件
    app.use(webpackMiddleware(compiler))
    // 启动 HTTP 服务，服务监听在 3000 端口
    app.listen(3000)
    ```
    webpackMiddleware 函数的返回结果是一个Express的中间件，该中间件具有以下功能：
    * 接受Webpack Compiler 实例输出的文件，但不会把文件输出到硬盘，而是保存在内存中。
    * 往Express app上注册路由，拦截HTTP收到的请求，根据请求路径响应对应的文件内容。
    由于是内存的文件系统，没有耗时的硬盘读写过程，数据的更新非常快。

2. Livereload和HMR  
    通过webpack-dev-middleware实现了监听修改 -> 触发构建流程的功能。但源码改动后，浏览器应该在何时获取重新编译后的资源，则需要Livereload和HMR实现。  
    1. Livereload  
        Livereload的原理是在浏览器和服务器之间创建WebSocket连接，服务器端在执行完动态编译之后发送reload事件至浏览器，浏览器接收到事件时候刷新整个页面。  

        ![Livereload模式](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/11.png?x-oss-process=image/resize,w_500)

    2. HMR  
        在开启webpack-dev-server模式下，webpack向构建输出的文件中注入了一项额外的功能模块——HMR Runtime。同时在服务器端也注入了对应的服务模块——HMR Server。与Livereload的实现方式类似的是，客户端与服务器之间也是通过WebSocket进行通信的。

        1. 修改源文件保存后，webpack监听到Filesystem Event事件并触发了重新构建行为。
        2. 构建完成后，webpack将模块变动信息传递给HMR Server。
        3. HMR Server通过WebSocket发送Push信息告知HMR Runtime需要更新客户端模块，HMR Runtime随后通过HTTP获取待更新模块的内容详情。
        4. 最终，HMR Runtime将更新的模块进行替换，在次过程中浏览器不会进行刷新。

        添加webpack-hot-middleware模块，并做如下修改才能支持模块热替换。
        1. 修改webpack.config.js文件，加入 HotModuleReplacementPlugin插件
            ```js
            module.exports = {
                entry: [
                    // 为了支持模块热替换，注入代理客户端
                    'webpack-hot-middleware/client',
                    // JS 执行入口文件
                    './src/main.js'
                ],
                ...
                plugins: [
                    // 为了支持模块热替换，生成 .hot-update.json 文件
                    new HotModuleReplacementPlugin(),
                ]
            };
            ```
        2. 修改HTTP服务代码，接入webpack-hot-middleware中间件
            ```js
            const webpackMiddleware = require('webpack-dev-middleware')
            ...
            // 为了支持模块热替换，响应用于替换老模块的资源
            app.use(require('webpack-hot-middleware')(compiler))
            ```
        
        ![HMR新生成的文件](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/StudyNotes/12.png?x-oss-process=image/resize,w_500)

#### 2. Mock服务
1. Mock的必要前提和发展进程  
    Mock的前提是前后端开发人员需要协定接口的规范细节，包括请求方法、入参、返回值等。  
    1. 假数据
        ```js
        let data = null
        // @todo 接口完成后删除
        const MockData = {
            a: 1
        }
        // @todo 接口完成后删除
        return data = MockData
        //请求开始前设置 loading 状态开启
        loading = true
        $.ajax({
            url: '/api',
            dataType: 'json',
            success: res => {
                console.log(res)
            }
        })
        ```
        **缺点：**  
        1. 如果前端逻辑中涉及大量接口调用，便会产生大量假数据对象以及注释。接口开发完成后，无用代码要全部删除，若一时疏忽忘记，你懂的...
        2. 数据不够真实，无法模拟接口请求流程及异常处理。导致与真实接口对接之前，遗漏许多逻辑。
        3. 另外一种亲身实践，血淋淋的教训。后端信誓旦旦地说接口已部署测试环境。我删掉了辛苦花费一万年写的假数据之后，接口又挂掉了，并且一个小时还没修复。我该怎么办！

    2. 客户端Mock  
        Mock进化的第二种形态是以Mock.js为代表的客户端Mock，工作原理是在客户端拦截JS发出的Ajax请求并返回由Mock.js创建的假数据。
        ```html
        <!-- @todo 接口完成后删除 -->
        <script src="/libs/js/mock.js"></script>
        <script>
        // 拦截 AJAX 请求并返回假数据
        // @todo 接口完成后删除
        Mock.mock('/api', {
            'code': 0,
            'errorMsg': 'success',
            'data': {
                'a': 1
            }
        })

        $.ajax({
            url: '/api',
            dataType: 'json',
            success: res => {
                console.log(res)
            }
        })
        </script>
        ```
        **缺点：**  
        1. Mock相关代码或文件仍然必须存在于业务代码中，而部署上线之前需要将其删除。
    
    3. Mock Server
        1. 结合webpack构建系统，使用Node将Mock Server集成到本地开发服务器中
        2. 使用三方Mock管理平台，Yapi、easyMock

TODO: 在本地开发服务器中集成Mock服务

### 五. 部署
**TODO: Gitlab + Jenkins + Ansible   ???**

### 六. 工作流

**TODO: Git + WebHook + Lint ???**

### 七. 前端工程化的未来
1. 容器技术
2. 微前端
3. serverless
4. 前端智能化

### 八. 引用

**《前端工程化 体系设计与实践》 周俊鹏**