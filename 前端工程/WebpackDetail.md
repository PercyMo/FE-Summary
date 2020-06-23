## webpack详解
TODO: 具体内容待填充...
### 一. webpack是什么
1. 和gulp，grunt区别

### 二. 基本使用

#### 1. npm script命令

#### 2. devtool 和 sourcemap

#### 3. webpack-dev-server 本地开发服务器

#### 4. 生产环境
`cross-env`设置环境变量的跨平台解决方案

#### 5. webpack-merge 多文件配置

#### 6. Loaders

1. style-loader（将css代码混合到js模块当中）

#### 7. plugins
1. Hot Module Replacement（HMR 模块热替换）
2. SplitChunks（抽取出通用代码模块）
3. MiniCssExtractPlugin （将css代码单独提取成文件）
4. ProvidePlugin （可以将jquery等常用类库设置全局变量）

#### 8. `import()`动态引入，懒加载
webpack将会把`import()`引入的模块单独打包成文件，按需引入

#### 9. 缓存
使用`[chunkhash]`

### 三. 流程详解

### 四. 最佳实践

### 五. 引用
[webpack原理（《深入浅出 webpack》）](https://webpack.wuhaolin.cn/5%E5%8E%9F%E7%90%86/5-1%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E6%A6%82%E6%8B%AC.html)

[Webpack 3，从入门到放弃 (官方文档内容)](https://segmentfault.com/a/1190000010871559#articleHeader24)