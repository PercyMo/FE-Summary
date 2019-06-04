### 2019.03
1. 【TODO】Postman 具体使用指南
2. 【TODO】Redis应用场景和基本使用，项目中使用redis做缓存的部分梳理下
3. 【TODO】Nginx的基本使用，百度云里有视频教程，网上搜下文章（原理大概了解下（对照下冬冬和周宁的配置conf，为什么2.0项目无法启动））
4. 【TODO】xtpl文档，模板引擎的工作原理，什么是模板引擎的runtime（顺便了解下jade）
5. 【TODO】js代码的性能测试，Benchmark.js
6. 【TODO】source_track，通过document.referrer 追踪用户注册来源，写入cookie，也算是一个业务实践
7. ~~通过$.append()方法添加一个script标签和通过原生js手动构建并添加一个标签有什么区别。jquery中通过append()插入script标签，其中的js代码可以正常执行，在zepto中则不行~~

### 2019.04
1. 【TODO】目前前端埋点相关的主流技术方案
      [**基于指令和混合的前端通用埋点方案](https://zhuanlan.zhihu.com/p/27659302)  
      [前端埋点统计方案思考](http://www.10tiao.com/html/780/201812/2650588763/1.html)  
      [神策系统vue埋点](https://my.oschina.net/u/3150903/blog/2086076?p=1)  
      [为什么你统计 PV 的方式是错的？](https://www.jianshu.com/p/84e617daf484)  
      [揭开JS无埋点技术的神秘面纱](http://unclechen.github.io/2018/06/24/%E6%8F%AD%E5%BC%80JS%E6%97%A0%E5%9F%8B%E7%82%B9%E6%8A%80%E6%9C%AF%E7%9A%84%E7%A5%9E%E7%A7%98%E9%9D%A2%E7%BA%B1/)  
      [美团点评前端无痕埋点实践](https://juejin.im/entry/58e8aa25a22b9d00589bd297)  
2. 【TODO】发现一种新的书写项目文档工具 gitbook
  [链接1](http://www.chengweiyang.cn/gitbook/basic-usage/README.html)
  [链接2](https://blog.csdn.net/lu_embedded/article/details/81100704)
3. 【TODO】使用charles做请求代理，手动修改数据响应类型，text/html  =>  image/gif
4. 【TODO】通过图片image标签src实现跨域的方案。。。
5. 【TODO】css变量 --bc-red: #f55555
  [链接](http://www.ruanyifeng.com/blog/2017/05/css-variables.html)
6. 【TODO】传统网站换肤功能实现方案
7. 【TODO】请求CORB警告 （跨域访问阻塞），情况：在三方类库中跨域访问.gif资源，服务器响应类型为text/html，造成CORB错误
  [链接](https://segmentfault.com/a/1190000016126079)
8. 【TODO】git工作流，追踪master分支代码变动
  [**git-workflow-tutorial.md](https://github.com/xirong/my-git/blob/master/git-workflow-tutorial.md)
  [开发与OP流程规范（git）](https://www.cnblogs.com/aylin/p/6042653.html)
  [阮一峰 Git 工作流程](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)
  [常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)
  [Git 工作流](https://juejin.im/post/5a014d5f518825295f5d56c7)
9. 【TODO】sentry用于项目错误监控
  [哨兵文件](https://docs.sentry.io/)
  [sentry使用实践](https://www.jianshu.com/p/66e00077fac3)
10. 【TODO】jq中ready的时机（DOMContentLoaded），原生实现
11. 【TODO】webpack 在开发环境中，项目build文件默认在内存中，默认启用文件端口等等
12. 【TODO】梳理整个项目启动的原理（有时间的时候，不急。2.0和3.0项目架构存在比较大差异，慢慢对比，如何支持多模板的编译，在开发环境当中，已经使用koa启动了项目，那么webpack在这一过程中如何发挥作用）
13. 【TODO】gitlab具体是什么，它和github的区别，比git多做了什么
14. 【TODO】sentry和gitlab等项目，它们允许用户配置不同的域名，然后接入自己的项目，开源的原理是什么

### 2019.05
1. 【TODO】在不刷新页面的情况下改变URL，模板详情弹窗的实现（html5新特性）
[如何在不刷新页面的情况下改变URL](https://zhuanlan.zhihu.com/p/22412047)
[在history中跳转](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API)
window.addEventListener('popstate', function() {})
2. 【TODO】path-to-regexp 路由匹配组件
3. ~~css文字渐变色~~

### 2019.06
1. 【TODO】httpOnly
2. 【TODO】host的配置原理