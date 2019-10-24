## Webpack优化
TODO: 具体内容待填充...
```
优化前：
Time: 1425ms

优化后：
Time: 1397ms
```
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

### 八. 引用
[webpack优化（《深入浅出 webpack》）](https://webpack.wuhaolin.cn/4%E4%BC%98%E5%8C%96/)

[webpack4.0打包优化策略](https://juejin.im/post/5abbc2ca5188257ddb0fae9b)