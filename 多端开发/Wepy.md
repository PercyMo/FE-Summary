## wepy官方框架

### Tip
1. 由于使用了wepy，初次运行小程序项目前，一定先全局安装WePY命令行工具
    ```
    npm install wepy-cli -g
    ```
2. 不可以把event监听放到minxin中

### 坑位
1. WePY 1.x 版本中，不支持在 repeat 的组件中去使用 props, computed, watch 等等特性。wepy的repeat与computed一起使用，会导致所有计算属性的值与循环里第一个item中计算属性的值相同，所以，在同一页面中n次（n>3）使用的公共部分，就索性不要提取组件了。问题太多。
2. 子组件通过props接受父组件传值时候，二级嵌套的数据是无法接收的
    ```html
    <child :propsData="obj.arrList"></child>    // 无法接收
    <child :propsData="arrList"></child>        // 可以接收
    ```
3. 使用$broadcast频繁广播事件，例如开了定时器。会非常影响小程序的性能，谨慎使用。
4. 小程序里修改data 里面的属性或者赋值都需要利用this.setdata()而wepy 基本就是利用this.属性即可。如果是异步返回或者更新dom需要this.$apply()触发脏值检测
5. wepy的mixin，与vue中的mixin执行顺序相反。wepy中，会先执行组件自身的，再执行mixin中的。vue中对于钩子函数，会先执行mixin中的，再执行组件自身的；vue中methods如果和mixin同名，那么只会执行自身的方法
6. 组件component没有正常的 onLoad、onShow 等页面事件，页面中设置 this.$broadcast('onShow', option)，组件监听事件则可以解决
7. 关闭开发工具提供的代码压缩选项，官方提醒：开启后，会导致真机computed, props.sync 等等属性失效。压缩功能可使用WePY提供的build指令代替