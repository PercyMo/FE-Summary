## 2019.03
1. 【TODO】Postman 具体使用指南
2. 【TODO】Redis应用场景和基本使用，项目中使用redis做缓存的部分梳理下
3. 【TODO】xtpl文档，模板引擎的工作原理，什么是模板引擎的runtime（顺便了解下jade）
4. 【TODO】js代码的性能测试，Benchmark.js
5. 【TODO】source_track，通过document.referrer 追踪用户注册来源，写入cookie，也算是一个业务实践
  [张鑫旭 document.referrer实践](https://www.zhangxinxu.com/wordpress/2017/02/js-page-url-document-referrer/)

## 2019.04
1. 【TODO】目前前端埋点相关的主流技术方案
    1. [**基于指令和混合的前端通用埋点方案](https://zhuanlan.zhihu.com/p/27659302)  
    2. [前端埋点统计方案思考](http://www.10tiao.com/html/780/201812/2650588763/1.html)  
    3. [神策系统vue埋点](https://my.oschina.net/u/3150903/blog/2086076?p=1)  
    4. [为什么你统计 PV 的方式是错的？](https://www.jianshu.com/p/84e617daf484)  
    5. [揭开JS无埋点技术的神秘面纱](http://unclechen.github.io/2018/06/24/%E6%8F%AD%E5%BC%80JS%E6%97%A0%E5%9F%8B%E7%82%B9%E6%8A%80%E6%9C%AF%E7%9A%84%E7%A5%9E%E7%A7%98%E9%9D%A2%E7%BA%B1/)  
    6. [美团点评前端无痕埋点实践](https://juejin.im/entry/58e8aa25a22b9d00589bd297)  
2. 【TODO】发现一种新的书写项目文档工具 gitbook
    1. [链接1](http://www.chengweiyang.cn/gitbook/basic-usage/README.html)
    2. [链接2](https://blog.csdn.net/lu_embedded/article/details/81100704)
3. 【TODO】使用charles做请求代理，手动修改数据响应类型，text/html  =>  image/gif
4. 【TODO】css变量 --bc-red: #f55555
  [链接](http://www.ruanyifeng.com/blog/2017/05/css-variables.html)
5. 【TODO】传统网站换肤功能实现方案
6. 【TODO】请求CORB警告 （跨域访问阻塞），情况：在三方类库中跨域访问.gif资源，服务器响应类型为text/html，造成CORB错误
  [Cross-Origin Read Blocking (CORB)](https://juejin.im/post/5cc2e3ecf265da03904c1e06)
  [链接](https://segmentfault.com/a/1190000016126079)
7. 【TODO】git工作流，追踪master分支代码变动
    1. [**git-workflow-tutorial.md](https://github.com/xirong/my-git/blob/master/git-workflow-tutorial.md)
    2. [开发与OP流程规范（git）](https://www.cnblogs.com/aylin/p/6042653.html)
    3. [阮一峰 Git 工作流程](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)
    4. [常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)
    5. [Git 工作流](https://juejin.im/post/5a014d5f518825295f5d56c7)
8. 【TODO】jq中ready的时机（DOMContentLoaded），原生实现
9. 【TODO】gitlab具体是什么，它和github的区别，比git多做了什么
10. 【TODO】sentry和gitlab等项目，它们允许用户配置不同的域名，然后接入自己的项目，开源的原理是什么

## 2019.05
1. 【TODO】path-to-regexp 路由匹配组件

## 2019.06
2. 【TODO】::v-deep
3. 【TODO】修改之后，执行 git add . && git commit --amend     然后 git push -f
4. 【TODO】JSON.stringify(vipItem.package, null, 2)
5. 【TODO】Vue 中 model: {
    prop: 'visible',
    event: 'visible-change'
  }
6. 【TODO】document.body.appendChild
  当同一个节点呗多次添加时，会先删除之前的节点然后，再添加
8. 【TODO】uuid，16进制转换10进制数字
9. 【TODO】// 下面两行是为了解决滚动太快时模板列表会先出现空白问题
  background-image: url(about:blank);
  background-attachment: fixed;

## 2019.07
1. 【TODO】基于IntersectionObserver的曝光统计

## 2019.08
1. 【TODO】从零开始写一个NPM包
2. 【TODO】前端学习的nginx知识
    1. [前端工程师学习Nginx入门篇](http://cnt1992.xyz/2016/03/18/simple-intro-to-nginx/)
    2. [Nginx与前端开发](https://juejin.im/post/5bacbd395188255c8d0fd4b2)
    3. [前端想要了解的Nginx](https://juejin.im/post/5cae9de95188251ae2324ec3)
    4. [8分钟带你深入浅出搞懂Nginx](https://zhuanlan.zhihu.com/p/34943332)
    5. [一个前端必会的 Nginx免费教程 (共11集）](http://jspang.com/posts/2018/10/05/nginx.html)
    6. 百度云里有视频教程
3. 【TODO】sentry & 前端错误监控
    1. [sentry使用实践](https://www.jianshu.com/p/66e00077fac3)
    2. [Sentry支持SourceMap指引](https://blog.fritx.me/?2017/07/sentry-sourcemap-guide)
    3. [超详细！搭建一个前端错误监控系统](https://zhuanlan.zhihu.com/p/51446011)
    4. [从源码分析sentry的错误信息收集](http://niexiaotao.cn/2018/08/18/%E4%BB%8E%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90sentry%E7%9A%84%E9%94%99%E8%AF%AF%E4%BF%A1%E6%81%AF%E6%94%B6%E9%9B%86/)
    5. [Sentry源码解析](https://hellogithub2014.github.io/2018/07/22/sentry-source-code/)
    6. [搭建前端监控系统（备选）用户行为统计和监控篇（如何快速定位线上问题）](https://www.cnblogs.com/warm-stranger/p/10209844.html)
4. 【TODO】几个最近发现的Linux命令行工具
    1. [每天学习一个命令：代码搜索工具](ack-grep http://einverne.github.io/post/2017/10/ack-grep.html)
    2. [Mac 下的 Top 和 Htop 指令](https://cnbin.github.io/blog/2015/06/19/mac-xia-de-top-he-htop-zhi-ling/)
5. 【TODO】Fetch & Ajax
6. 【TODO】babel-polyfill