## Webpack优化
TODO: 具体内容待填充...
### 一. 缩小文件搜索范围
1. 优化`Loader`配置
    ```js
    module: {
        rules: [
            {
                // 如果项目源码中只有 js 文件就不要写成 /\.jsx?$/，提升正则表达式性能
                test: /\.js$/,
                // babel-loader 支持缓存转换出的结果，通过 cacheDirectory 选项开启
                use: ['babel-loader?cacheDirectory'],
                // 只对项目根目录下的 src 目录中的文件采用 babel-loader
                include: path.resolve(__dirname, 'src')
            }
        ]
    }
    ```

2. 优化`resolve.modules`配置
    ```js
    resolve: {
        // 使用绝对路径指明第三方模块存放的位置，以减少搜索步骤
        modules: [path.resolve(__dirname, 'node_modules')]
    }
    ```

3. 优化`resolve.mainFields`配置
    ```js
    resolve: {
        // 只采用 main 字段作为入口文件描述字段，以减少搜索步骤
        mainFields: ['main']
    }
    ```

4. 优化`resolve.alias`配置
    ```js
    resolve: {
        // 使用 alias 把导入 react 的语句换成直接使用单独完整的 react.min.js 文件，
        // 减少耗时的递归解析操作
        alias: {
            'react': path.resolve(__dirname, './node_modules/react/umd/react.production.min.js') // react16
        }
    }
    ```

5. 优化`resolve.extensions`配置
    ```js
    resolve: {
        // 尽可能的减少后缀尝试的可能性
        extensions: ['js'],
    }
    ```

6. 优化`module.noParse`配置
    ```js
    module: {
        // 独完整的 `react.min.js` 文件就没有采用模块化，忽略对 `react.min.js` 文件的递归解析处理
        noParse: [/react\.min\.js$/],
    }
    ```

### 二. DllPlugin 动态链接库

### 三. HappyPack
```js
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HappyPack = require('happypack');

module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                // 把对 .js 文件的处理转交给 id 为 babel 的 HappyPack 实例
                use: ['happypack/loader?id=babel']
            },
            {
                // 把对 .css 文件的处理转交给 id 为 css 的 HappyPack 实例
                test: /\.css$/,
                use: ExtractTextPlugin.extract({
                    use: ['happypack/loader?id=css']
                }),
            },
        ]
    },
    plugins: [
        new HappyPack({
            // 用唯一的标识符 id 来代表当前的 HappyPack 是用来处理一类特定的文件
            id: 'babel',
            // 如何处理 .js 文件，用法和 Loader 配置中一样
            loaders: ['babel-loader?cacheDirectory'],
            // ... 其它配置项
        }),
        new HappyPack({
            id: 'css',
            // 如何处理 .css 文件，用法和 Loader 配置中一样
            loaders: ['css-loader'],
        }),
        new ExtractTextPlugin({
            filename: `[name].css`,
        }),
    ],
};
```
使用多进程处理`loader`编译，由`happypack`分解任务，管理线程。

### 四. 使用 ParallelUglifyPlugin
替代`UglifyJSPlugin`多进程压缩js

### 五. webpack-dev-server
1. 监听文件修改，使用自动刷新
2. 开启模块热替换

### 六. 区分环境
通过环境变量`process.env.NODE_ENV`，区分开发和生产环境，分别配置

### 七. 压缩代码
1. 压缩`JS`
2. 压缩`ES6`  
    `ES6`相对`ES5`代码量更少，若浏览器适配相对宽松，可以考虑适当保留`ES6`语法
3. 使用`OptimizeCSSAssetsPlugin`压缩css

### 八. CDN加速

### 九. Tree Shaking
依赖静态的ES6模块化语法，剔除JS中用不上的死代码。

### 十. 提取公共代码
`CommonsChunkPlugin`，`splitChunks`

### 十一. 按需加载
1. 使用webpack的`import()`语法，将模块单独抽离文件，在适当时机使用代码按需引入。实现模块的按需加载。
2. React，Vue的都提供了路由懒加载，实质也是按需加载。

### 十二. 使用Prepack
在保持运行结果一致的情况下，改变源代码的运行逻辑，输出性能更高的JS代码。实际上，Prepack就是一个部分求值器，编译代码时提前将计算结果放到编译后的代码中，而不是在代码运行时才去求值。
```js
import React, {Component} from 'react';
import {renderToString} from 'react-dom/server';

function hello(name) {
  return 'hello ' + name;
}

class Button extends Component {
  render() {
    return hello(this.props.name);
  }
}

console.log(renderToString(<Button name='webpack'/>));
```
被 Prepack 转化后竟然直接输出如下：
```js
console.log("hello webpack");
```

### 十二. 开启Scope Hoisting（作用域提升）
分析模块之间的依赖关系，尽可能把打散的模块合并到一个函数中去，但前提是不能造成代码冗余，因此只有那些被引用了一次的模块才能被合并。

### 十三. 输出分析
`Webpack Analyse`，`webpack-bundle-analyzer`

### 十四. 优化总结配置
1. 侧重优化开发体验的配置文件`webpack.config.js`
    ```js
    const path = require('path');
    const CommonsChunkPlugin = require('webpack/lib/optimize/CommonsChunkPlugin');
    const {AutoWebPlugin} = require('web-webpack-plugin');
    const HappyPack = require('happypack');

    // 自动寻找 pages 目录下的所有目录，把每一个目录看成一个单页应用
    const autoWebPlugin = new AutoWebPlugin('./src/pages', {
    // HTML 模版文件所在的文件路径
    template: './template.html',
    // 提取出所有页面公共的代码
    commonsChunk: {
        // 提取出公共代码 Chunk 的名称
        name: 'common',
    },
    });

    module.exports = {
    // AutoWebPlugin 会找为寻找到的所有单页应用，生成对应的入口配置，
    // autoWebPlugin.entry 方法可以获取到生成入口配置
    entry: autoWebPlugin.entry({
        // 这里可以加入你额外需要的 Chunk 入口
        base: './src/base.js',
    }),
    output: {
        filename: '[name].js',
    },
    resolve: {
        // 使用绝对路径指明第三方模块存放的位置，以减少搜索步骤
        // 其中 __dirname 表示当前工作目录，也就是项目根目录
        modules: [path.resolve(__dirname, 'node_modules')],
        // 针对 Npm 中的第三方模块优先采用 jsnext:main 中指向的 ES6 模块化语法的文件，使用 Tree Shaking 优化
        // 只采用 main 字段作为入口文件描述字段，以减少搜索步骤
        mainFields: ['jsnext:main', 'main'],
    },
    module: {
        rules: [
        {
            // 如果项目源码中只有 js 文件就不要写成 /\.jsx?$/，提升正则表达式性能
            test: /\.js$/,
            // 使用 HappyPack 加速构建
            use: ['happypack/loader?id=babel'],
            // 只对项目根目录下的 src 目录中的文件采用 babel-loader
            include: path.resolve(__dirname, 'src'),
        },
        {
            test: /\.js$/,
            use: ['happypack/loader?id=ui-component'],
            include: path.resolve(__dirname, 'src'),
        },
        {
            // 增加对 CSS 文件的支持
            test: /\.css$/,
            use: ['happypack/loader?id=css'],
        },
        ]
    },
    plugins: [
        autoWebPlugin,
        // 使用 HappyPack 加速构建
        new HappyPack({
        id: 'babel',
        // babel-loader 支持缓存转换出的结果，通过 cacheDirectory 选项开启
        loaders: ['babel-loader?cacheDirectory'],
        }),
        new HappyPack({
        // UI 组件加载拆分
        id: 'ui-component',
        loaders: [{
            loader: 'ui-component-loader',
            options: {
            lib: 'antd',
            style: 'style/index.css',
            camel2: '-'
            }
        }],
        }),
        new HappyPack({
        id: 'css',
        // 如何处理 .css 文件，用法和 Loader 配置中一样
        loaders: ['style-loader', 'css-loader'],
        }),
        // 4-11提取公共代码
        new CommonsChunkPlugin({
        // 从 common 和 base 两个现成的 Chunk 中提取公共的部分
        chunks: ['common', 'base'],
        // 把公共的部分放到 base 中
        name: 'base'
        }),
    ],
    watchOptions: {
        // 4-5使用自动刷新：不监听的 node_modules 目录下的文件
        ignored: /node_modules/,
    }
    };
    ```

2. 侧重优化输出质量的配置文件`webpack-dist.config.js`
    ```js
    const path = require('path');
    const DefinePlugin = require('webpack/lib/DefinePlugin');
    const ModuleConcatenationPlugin = require('webpack/lib/optimize/ModuleConcatenationPlugin');
    const CommonsChunkPlugin = require('webpack/lib/optimize/CommonsChunkPlugin');
    const ExtractTextPlugin = require('extract-text-webpack-plugin');
    const {AutoWebPlugin} = require('web-webpack-plugin');
    const HappyPack = require('happypack');
    const ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');

    // 自动寻找 pages 目录下的所有目录，把每一个目录看成一个单页应用
    const autoWebPlugin = new AutoWebPlugin('./src/pages', {
    // HTML 模版文件所在的文件路径
    template: './template.html',
    // 提取出所有页面公共的代码
    commonsChunk: {
        // 提取出公共代码 Chunk 的名称
        name: 'common',
    },
    // 指定存放 CSS 文件的 CDN 目录 URL
    stylePublicPath: '//css.cdn.com/id/',
    });

    module.exports = {
    // AutoWebPlugin 会找为寻找到的所有单页应用，生成对应的入口配置，
    // autoWebPlugin.entry 方法可以获取到生成入口配置
    entry: autoWebPlugin.entry({
        // 这里可以加入你额外需要的 Chunk 入口
        base: './src/base.js',
    }),
    output: {
        // 给输出的文件名称加上 Hash 值
        filename: '[name]_[chunkhash:8].js',
        path: path.resolve(__dirname, './dist'),
        // 指定存放 JavaScript 文件的 CDN 目录 URL
        publicPath: '//js.cdn.com/id/',
    },
    resolve: {
        // 使用绝对路径指明第三方模块存放的位置，以减少搜索步骤
        // 其中 __dirname 表示当前工作目录，也就是项目根目录
        modules: [path.resolve(__dirname, 'node_modules')],
        // 只采用 main 字段作为入口文件描述字段，以减少搜索步骤
        mainFields: ['jsnext:main', 'main'],
    },
    module: {
        rules: [
        {
            // 如果项目源码中只有 js 文件就不要写成 /\.jsx?$/，提升正则表达式性能
            test: /\.js$/,
            // 使用 HappyPack 加速构建
            use: ['happypack/loader?id=babel'],
            // 只对项目根目录下的 src 目录中的文件采用 babel-loader
            include: path.resolve(__dirname, 'src'),
        },
        {
            test: /\.js$/,
            use: ['happypack/loader?id=ui-component'],
            include: path.resolve(__dirname, 'src'),
        },
        {
            // 增加对 CSS 文件的支持
            test: /\.css$/,
            // 提取出 Chunk 中的 CSS 代码到单独的文件中
            use: ExtractTextPlugin.extract({
            use: ['happypack/loader?id=css'],
            // 指定存放 CSS 中导入的资源（例如图片）的 CDN 目录 URL
            publicPath: '//img.cdn.com/id/'
            }),
        },
        ]
    },
    plugins: [
        autoWebPlugin,
        // 4-14开启ScopeHoisting
        new ModuleConcatenationPlugin(),
        // 4-3使用HappyPack
        new HappyPack({
        // 用唯一的标识符 id 来代表当前的 HappyPack 是用来处理一类特定的文件
        id: 'babel',
        // babel-loader 支持缓存转换出的结果，通过 cacheDirectory 选项开启
        loaders: ['babel-loader?cacheDirectory'],
        }),
        new HappyPack({
        // UI 组件加载拆分
        id: 'ui-component',
        loaders: [{
            loader: 'ui-component-loader',
            options: {
            lib: 'antd',
            style: 'style/index.css',
            camel2: '-'
            }
        }],
        }),
        new HappyPack({
        id: 'css',
        // 如何处理 .css 文件，用法和 Loader 配置中一样
        // 通过 minimize 选项压缩 CSS 代码
        loaders: ['css-loader?minimize'],
        }),
        new ExtractTextPlugin({
        // 给输出的 CSS 文件名称加上 Hash 值
        filename: `[name]_[contenthash:8].css`,
        }),
        // 4-11提取公共代码
        new CommonsChunkPlugin({
        // 从 common 和 base 两个现成的 Chunk 中提取公共的部分
        chunks: ['common', 'base'],
        // 把公共的部分放到 base 中
        name: 'base'
        }),
        new DefinePlugin({
        // 定义 NODE_ENV 环境变量为 production 去除 react 代码中的开发时才需要的部分
        'process.env': {
            NODE_ENV: JSON.stringify('production')
        }
        }),
        // 使用 ParallelUglifyPlugin 并行压缩输出的 JS 代码
        new ParallelUglifyPlugin({
        // 传递给 UglifyJS 的参数
        uglifyJS: {
            output: {
            // 最紧凑的输出
            beautify: false,
            // 删除所有的注释
            comments: false,
            },
            compress: {
            // 在UglifyJs删除没有用到的代码时不输出警告
            warnings: false,
            // 删除所有的 `console` 语句，可以兼容ie浏览器
            drop_console: true,
            // 内嵌定义了但是只用到一次的变量
            collapse_vars: true,
            // 提取出出现多次但是没有定义成变量去引用的静态值
            reduce_vars: true,
            }
        },
        }),
    ]
    };
    ```

### 十五. 引用
[webpack优化（《深入浅出 webpack》）](https://webpack.wuhaolin.cn/4%E4%BC%98%E5%8C%96/)

[webpack4.0打包优化策略](https://juejin.im/post/5abbc2ca5188257ddb0fae9b)