## 微信小程序性能优化

### 一. 怎么定义高性能？
* FCP：白屏加载结束
* FMP：首屏渲染完成
* TTI：所有内容加载完成

### 二. 白屏时间优化

### 三. 渲染性能优化
#### 1. 长列表优化——虚拟列表
1. 结构改造：外层容器，监听滚动，触发更新计算
2. 卡片高度计算：根据卡片内容类型，计算卡片精确高度
3. 白屏优化：未出现在可视区的部分为空白节点，通过卡片骨架，优化滚动视觉效果

#### 2. 低端机降级
* 划分标准：根据发型年份，机型，系统，划分高低端机（33%低端+67%高端）
* 优化内容
    * 轮播图改单图
    * 动图改静图
    * 大图改小图
    * 去除跨模块关联
    * 去除时间关联

### 四. 引用
[京喜小程序的高性能打造之路](https://juejin.cn/post/6844904104171438088)