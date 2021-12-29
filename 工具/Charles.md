## Charles

### 一. 修改 Request 和 Response 数据

#### 1. 通过 Breakpoints 断点功能
1. 选中某个网络请求 -> 右键 -> Breakpoints  

    ![Breakpoints](http://img.vanilla.ink/me/webproject/FE-Summary/Tools/Charles/01.png?x-oss-process=image/resize,w_600)

2. Charles工具中 Proxy -> Breakpoint Settings  

    这里可以修改断点信息，例如 query 中存在时间戳这种每次请求都变化的参数，无法命中断点。就可以在这里把时间戳参数去掉。  

    ![Breakpoint Settings](http://img.vanilla.ink/me/webproject/FE-Summary/Tools/Charles/02.png?x-oss-process=image/resize,w_600)

    命中断点后，`Edit Response` 修改响应数据。点击 `Execute` 执行下一步，就可以看到被修改的请求已经生效了。

### 二. Mac 上配置抓包
[Mac下用Charles实现Android http和https抓包](https://blog.csdn.net/luochoudan/article/details/72801573)  

[Mac上使用Charles抓包](https://zhuanlan.zhihu.com/p/26182135)  