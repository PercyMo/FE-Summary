## mpvue类vue框架

### Tip
1. 原生组件`<button>`，不能再使用`bindgetuserinfo`绑定回调，只能使用`@getuserinfo`，且返回值不同于原生，旧返回值被进一步封装在`mp: {}`对象下，使用时一定注意。

2. 同一个页面被多次打开所引发的问题

    ```js
    // /src/util/Hack.js
    const pageDatas = {}

    export default {
        install(_Vue) {
            // 添加全局方法或属性
            _Vue.prototype.$isPage = function isPage() {
            return this.$mp && this.$mp.mpType === 'page'
            }
            
            _Vue.prototype.$pageId = function pageId() {
            let pid = null
            try {
                pid = this.$isPage() ? this.$mp.page.__wxWebviewId__ : null
            } catch (e) { }
            return pid
            }
            
            // 注入组件
            _Vue.mixin({
            
            methods: {
                stashPageData() {
                return { data: { ...this.$data } }
                },
                restorePageData(oldData) {
                Object.assign(this.$data, oldData.data)
                },
            },
            
            onLoad() {
                if (this.$isPage()) {
                // 新进入页面
                Object.assign(this.$data, this.$options.data())
                }
            },
            
            onUnload() {
                if (this.$isPage()) {
                // 退出页面，删除数据
                delete pageDatas[this.$pageId()]
                this.$needReloadPageData = true
                }
            },
            
            onHide() {
                if (this.$isPage()) {
                // 将要隐藏时，备份数据
                pageDatas[this.$pageId()] = this.stashPageData()
                }
            },
            
            onShow() {
                if (this.$isPage()) {
                // 如果是后退回来的，拿出历史数据来设置data
                if (this.$needReloadPageData) {
                    const oldData = pageDatas[this.$pageId()]
                    if (oldData) {
                    this.restorePageData(oldData)
                    }
                    this.$needReloadPageData = false
                }
                }
            },
            
            })
        },
    }
    ```
    ```js
    // src/main.js
    import Vue from 'vue'
    import App from './App'
    import Hack from './utils/Hack'

    Vue.config.productionTip = false
    App.mpType = 'app'
    Vue.use(Hack)

    const app = new Vue(App)
    app.$mount()
    ```
3. `scroll-view` 组件的一些问题

    不推荐使用这个组件，也不是mpvue的问题，就是这个组件本身就不太好用

    事件触发不了的问题 在mpvue中使用下面的一些事件的时候是不会触发的
    * `bindscrolltolower`
    * `bindscrolltoupper`
    * `bindscroll`  
    解决方法就是把bind去掉
    * `scrolltolower`
    * `scrolltoupper`
    * `scroll`

    然而，当 `scroll-view` 高度为 100% 的时候 `bindscrolltolower` 又不执行了，解决方法是给外面的父元素加一些限制

    ```html
    <div class="cat-scroll">
        <scroll-view style="height: 100%; width: 100%">
        </<scroll-view>
    </div>
    ```
    ```css
    .cat-scroll {
        position: absolute;
        left: 0;
        right: 0;
        bottom: 0;
        top: 0;
        overflow: hidden;
    }
    ```
### 坑位
1. input输入框fix定位不准问题

    解决没有意义，建议绕过该问题，更改设计。
