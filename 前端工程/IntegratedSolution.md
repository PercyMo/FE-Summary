## 前端工程化概述

### 一. 什么是工程化
#### 1. 定义
* 工程化即系统化、模块化、规范化的一个过程。
* 前端是一种技术问题较少、工程问题较多的软件开发领域。

#### 2. 解决什么问题
以工程化方法和工具，提高编码、测试、维护阶段的生产效率。
> 如果你没有维护过一个超过3000行的JavaScript代码文件，可能会无法理解工程化带来的收益。 ——邵充

### 二. 前端发展史
1. **简单明快的早期时代**  
    Web1.0，适合小项目，不分前后端，页面由JSP、PHP等在服务端生成，浏览器负责展现。

2. **后端为主的MVC时代**  
    为了降低复杂度，以后端为出发点，有了Web Server层的架构升级，比如Structs、Spring MVC等。  

    ![MVC](https://vani.oss-cn-beijing.aliyuncs.com/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/IntegratedSolution/01.png?x-oss-process=image/resize,w_500)

> **前两个阶段存在的问题**
> 1. 前端开发重度依赖开发环境
> 2. 前后端职责依旧纠缠不清，可维护性越来越差

3. **Ajax带来的SPA时代**  
    2005年，ajax正式提出，前端进入SPA（Single Page Application 单页面应用）时代。  

    ![web2.0](https://vani.oss-cn-beijing.aliyuncs.com/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/IntegratedSolution/02.png?x-oss-process=image/resize,w_500)

    Ajax时代面临的挑战：
        1. 前后端接口的约定
        2. 前端开发复杂度的控制。SPA应用大多以功能交互型为主，大量JS代码的组织，与View层的绑定等，都是不容易的事情。

4. **前端为主的MVC、MV\*时代**  
    为了降低前端开发复杂度，Backbone、EmberJS、KnockoutJS、AngularJS、React、Vue等大量前端框架涌现。（组件化开发以及合理分层）

    ![SPA](https://vani.oss-cn-beijing.aliyuncs.com/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/IntegratedSolution/03.png?x-oss-process=image/resize,w_500)

5. **Node带来的全栈时代**  
    随着NodeJS的兴起，为前端带来了一种新的开发模式。业界比较出名的实践是，阿里巴巴的中途岛计划（Midway，基于eggJS的二开）。  

    ![全栈时代](https://vani.oss-cn-beijing.aliyuncs.com/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/IntegratedSolution/04.png?x-oss-process=image/resize,w_400)

> **后两个阶段的优势**
> 1. 前后端职责很清晰
> 2. 前端开发的复杂度可控，通过合理的分层，让项目更可维护
> 3. 部署相对独立，产品体验可以快速改进

### 三. 前端工程化要解决什么
#### 1. 使用合适的前端技术和框架，提高生产效率，降低维护难度
**目标**：职责分离、降低耦合，增强代码的可读性、维护性和测试性。
> 模块化开发最大的价值是分治。不管将来你是否要复用某段代码，你都有充分的理由将其分治为一个模块。
1. 采用模块化方式组织代码
    * JS模块化：AMD、CommonJS、UMD、ES6 Module
    * CSS模块化：less、sass、stylus、postCSS、css module（inport/mixin特性支持）

2. 采用组件化的编程思想，处理UI层
    React、Vue、Angular  

    ![组件化](https://vani.oss-cn-beijing.aliyuncs.com/me/webproject/FE-Summary/%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B/IntegratedSolution/05.png?x-oss-process=image/resize,w_500)

    前端组件化开发理念：  

    1. 页面上每一个**独立的**可视/可交互区域为一个组件
    2. **每个组件对应一个工程目录**，组件所需的各种资源都在这个目录下**就近维护**
    3. 由于组件具有独立性，因此组件和组件之间可以**自由组合**
    4. 页面只不过是组件的容器，负责组合组件形成功能完整的界面
    5. 当不需要某个组件，或者想替换组件时，可以整个目录删除/替换

    基于上面的模块化开发，整个前端项目可以划分为这么几种开发概念：
    | 名称 | 说明 | 举例 |
    |:- |:- |:- |
    | JS模块 | 独立的算法和数据单元 | 浏览器环境检测(detect)，网络请求(ajax)，应用配置(config)，DOM操作(dom)，工具函数(utils)，以及组件里的JS单元 |
    | CSS模块 | 独立的功能性样式单元 | 栅格系统(grid)，字体图标(icon-fonts)，动画样式(animate)，以及组件里的css单元 |
    | UI组件 | 独立的可视/可交互功能单元 | 页头(header)，页尾(footer)，导航栏(nav)，选项卡(tabs)，表单元素(input)... |
    | 页面 | 前端这种GUI软件的界面状态，是UI组件的容器 | 首页(index)，列表页(list)，用户管理(user) |
    | 应用 | 整个项目或整个站点被称之为应用，由多个页面组成 |

3. 将数据层分离管理
    redux、mbox、vuex

4. 使用面向对象或者函数编程的方式组织架构

#### 2. 项目构建
1. 前端构建工具  
    gulp、grunt、webpack、Rollup、Parcel
2. JS编译工具  
    Babel、Browserify

#### 3. 限定规范
1. 统一编码规范
    * 目录结构，文件命名规范
    * 编码规范，eslint、stylelint

2. 开发流程的规范
    * 使用敏捷，增强开发进度管理和控制
    * 应对各项风险，需求变更等
    * code review机制
    * UAT提升发布的需求的质量
3. 前后端接口规范，其他文档规范
    * 数据mock

#### 4. 增强可测试性
1. 采用tdd的编程思想，引入单元测试  
    karma + mocha + chai、jest、ava
2. 使用各种调试工具  
    web devtools、Charles

#### 5. 版本控制
1. 使用git版本控制工具
2. Git分支管理
3. Commit描述规范，例如：task-number + task描述
4. 创建merge request，code review完毕之后方可合并代码

#### 6. 持续集成交付
1. jenkins

### 四. 基础建设示例
1. xx-boilerplate 脚手架项目
2. xx-tools 通用工具库
3. xx-lib 通用类库
4. xx-component 公共组件库
5. eslint-config eslint规范
6. stylelint-config stylelint规范
7. xx-docs 各种规范文档
8. xx-style PC端样式库

### 五. 鸡汤
> 如何选型技术、如何定制规范、如何分治系统、如何优化性能、如何加载资源，当你从切图转变为思考这些问题的时候，我想说：
> 你好，工程师！

### 六. 引用
[前端工程——基础篇](https://github.com/fouber/blog/issues/10#)

[前端工程化概述](https://juejin.im/post/5ac9c6f451882555677ed301)
