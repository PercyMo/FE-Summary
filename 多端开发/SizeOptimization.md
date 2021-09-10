## 微信小程序体积优化

### 一. 常见瘦身方式
1. 代码分包
2. 通过代码分析工具找到体积占用明显的文件，针对性优化；将并不通用的组件或方法，移动到子包中；移除未使用的内容  
    ![依赖分析：移除未使用的内容](http://img.vanilla.ink/me/webproject/FE-Summary/MultiEnd/SizeOptimization/02.png?x-oss-process=image/resize,w_500)
3. 避免使用本地资源
4. 所有类型都进行压缩和注释清理
5. 继承属性  
    color/line-height/font-size/font-family
6. 样式补全  
    开发者自己补全 vs 开启编译补全
7. 样式命名优化  
    类似于 UglifyJs，使用 cssModules 和 单字母编排  
    ![样式命名优化](http://img.vanilla.ink/me/webproject/FE-Summary/MultiEnd/SizeOptimization/01.png?x-oss-process=image/resize,w_450)

### 二. 实践

### 三. 引用
[京喜小程序首页瘦身实践——给小程序再减重30%的秘密](https://jelly.jd.com/article/5fbfa6e9cff6b301458392c3)