## weapp-thinking

### 前言
小程序遇到问题可以先去[小程序官方社区](https://developers.weixin.qq.com/)和[小程序官方仓库](https://github.com/Tencent/wepy/issues)找下有没有类似问题。

### 原理
1. data数据绑定原理

    在针对数据绑定做优化时，需要先了解小程序的运行机制。因为视图层与逻辑层的完全分离，所以二者之间的通信全都依赖于 WeixinJSBridge 实现。如：
    * 开发者工具中是基于 `window.postMessage`
    * IOS中基于 `window.webkit.messageHandlers.invokeHandler.postMessage`
    * Android中基于 `WeixinJSCore.invokeHandler`
    
    因此数据绑定方法this.setData也如此，频繁的数据绑定就增加了通信的成本。再来看看this.setData究竟做了哪些事情。基于开发者工具的代码，单步调试大致还原出完整的流程，以下是还原后的代码：
    ```javascript
    /*
    setData 主流程精简还原，并非完整主流程，内有注释
    */
    function setData (obj) {
        if (typeof(obj) !== 'Object') {
            console.log('类型错误'); // 并没有预期中的return;
        }
        let type = 'appDataChange';

        // u.default.emit(e, this.__wxWebviewId__) 代码还原
        let e = [type, {
                    data: {data: list}, 
                    options: {timestamp: +new Date()}
                },
                [0] // this.__wxWebviewId__
        }];

        // WeixinJSBridge.publish.apply(WeixinJSBridge, e); 代码还原
        var datalength = JSON.stringify(e.data).length;  // 第一次 JSON.stringify
        if (datalength > AppserviceMaxDataSize) { // AppserviceMaxDataSize === 1048576
            console.error('已经超过最大长度');
            return;
        }

        if (type === 'appDataChange' || type === 'pageInitData' || type === '__updateAppData') {

            // sendMsgToNW({appData: __wxAppData, sdkName: "send_app_data"}) 代码还原
            __wxAppData = {
                'pages/page1/page1': alldata
            }
            e = { appData: __wxAppData, sdkName: "send_app_data" }

            var postdata = JSON.parse(JSON.stringify(e)); // 第二次 JSON.stringify 第一次 JSON.parse
            window.postMessage({
                postdata
            }, "*");
        }


        // sendMsgToNW({appData: __wxAppData, sdkName: "send_app_data"}) 代码还原
        e = {
            eventName: type,
            data: e[1],
            webviewIds: [0],
            sdkName: 'publish'
        };

        var postdata = JSON.parse(JSON.stringify(e));  // 第三次 JSON.stringify 第二次 JSON.parse
        window.postMessage({
            postdata
        }, "*");
    }
    ```
    setData 运行的流程如下：

    ![name](https://note.youdao.com/yws/public/resource/0c7e34c500438947463771cc1073655a/xmlnote/WEBRESOURCE19db5ee84229c8180ffba53963a0c162/552 '描述')

    从上面代码以及流程图中可以看出，在一次setData({a: 1})作时，会进行三次 JSON.stringify，二次JSON.parse以及两次window.postMessage操作。并且在第一次window.postMessage时，并不是单单只处理传递的{a:1}，而是处理当前页面的所有 data 数据。因此可想而知每次setData操作的开销是非常大的，只能通过减少数据量，以及减少setData操作来规避。

### Tip
1. button按钮默认的样式一般无法满足需求（带一圈很细的边线，由于IDE不显示伪类，一开始找了很久没找到那条细线怎么来的，去还去不掉，比较坑），可以把按钮样式统一重置
    ```css
    button {
        padding: 0;
        background: #fff;
        line-height: 0;
        &::after {
            border-color: transparent;
        }
    }
    .button-hover {
        background: #fff;
    }
    ```
2. 微信小程序跨页面通信解决思路。在网上总结了下几种方案。最终选择了京东的方案 发布／订阅模式

    [京东凹凸实验室的一篇文章](https://aotu.io/notes/2017/01/19/wxapp-event/index.html)

    [某位大牛的方案weapp-event，京东也是参考了这篇文章](https://github.com/dannnney/weapp-event)
3. 在app.js中注册globalData供所有页面取用。例如可以把systemInfo直接注册到globalData中，这样就不用在每个页面都获取一遍
4. 【性能优化】小程序运行在微信平台，而且可能和众多小程序“共享运行内存”，可想而知，单个小程序的性能极可能遇到瓶颈而Crash或被微信主动销毁！
    因此，尽量减少页面和组件的数量。对于相似的内容，尽量合并。通过不同的State来改变页面和组件的不同展现
5. 小程序编译时，点上代码自动补全，适配老版本浏览器，尤其是老iphone及老IOS版本
6. 【性能优化】不同于传统HTML页面。双线程下的界面渲染，小程序的逻辑层和渲染层是分开的两个线程。在渲染层，宿主环境会把WXML转化成对应的JS对象，在逻辑层发生数据变更的时候，我们需要通过宿主环境提供的setData方法把数据从逻辑层传递到渲染层，再经过对比前后差异，把差异应用在原来的Dom树上，渲染出正确的UI界面。

    因此，与界面渲染无关的静态数据不宜放到data中管理，data中存储的数据应该尽可能精简。例如在feed流等类似组件中，过多的冗余数据，会导致在几次分页之后，setdata因数据超出限制而失败。而且数据过多也会很大程度降低页面性能。

    同时，不要过于频繁调用setData，应考虑将多次setData合并成一次setData调用

    提升数据更新性能方式的代码示例
    ```javascript
    Page({
        onShow: function() {
            // 不要频繁调用setData
            this.setData({ a: 1 })
            this.setData({ b: 2 })
            // 绝大多数时候可优化为
            this.setData({ a: 1, b: 2 })

            // 不要设置不在界面渲染时使用的数据，并将界面无关的数据放在data外
            this.setData({
            myData: {
                a: '这个字符串在WXML中用到了',
                b: '这个字符串未在WXML中用到，而且它很长…………………………'
            }
            })
            // 可以优化为
            this.setData({
            'myData.a': '这个字符串在WXML中用到了'
            })
            this._myData = {
            b: '这个字符串未在WXML中用到，而且它很长…………………………'
            }
        }
    })
    ```
    
7. 在小程序里，每个 Page 都是一个模块，有着独立的作用域。
8. 【内容审核】imgSecCheck，校验一张图片是否含有违法违规内容。应用场景举例：1）图片智能鉴黄：涉及拍照的工具类应用(如美拍，识图类应用)用户拍照上传检测；电商类商品上架图片检测；媒体类用户文章里的图片检测等；2）敏感人脸识别：用户头像；媒体类用户文章里的图片检测；社交类用户上传的图片检测等。

    msgSecCheck，检查一段文本是否含有违法违规内容。应用场景举例：用户个人资料违规文字检测；媒体新闻类用户发表文章，评论内容检测；游戏类用户编辑上传的素材(如答题类小游戏用户上传的问题及答案)检测等。
    
    频率限制：单个 appId 调用上限为 2000 次/分钟，1,000,000 次/天
9. 【体验优化】为避免一些页面不必要的加载白屏，可以在前置页面将一些有用的字段带到当前页，进行首次渲染（列表页的某些数据--> 详情页），没有数据的模块可以进行骨架屏的占位，使用户不会等待的很焦虑，甚至走了
10. 小程序中遮罩层的滚动穿透问题

    (1)如果弹出层没有滚动事件，就直接在蒙板上加`catchtouchmove="true"`

    (2)如果弹出层有滚动事件，那么在弹出层出现的时候给底部的containerView加上一个class, 消失的时候移除。(一个比较严重的问题，安卓正常，在iphone下，页面滚动会有明显卡顿。最终放弃)
    ```css
    .container {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        z-index: 0;
        overflow-y: scroll;
        &.no_scroll {
            overflow: hidden;
        }
    }
    ```
11. 父组件调用子组件的方法

    `vue`会给子组件添加一个`ref`属性，通过`this.$refs.ref`的值便可以获取到该子组件，然后便可以调用子组件中的任意方法，例如：
    ```
    //子组件
    <bar ref="bar"></bar>
    //父组件
    this.$ref.bar.子组件的方法
    ```
    小程序是给子组件添加`id`或者`class`，然后通过`this.selectComponent`找到子组件，然后再调用子组件的方法,示例：
    ```
    //子组件
    <bar id="bar"></bar>
    // 父组件
    this.selectComponent('#id').syaHello()
    ```
12. 微信小程序`<view>`组件不解析换行符，因此对于描述性带有换行符的文字，使用`<text>`标签
13. 信小程序保存图片到系统相册

    图片的保存，这个图片保存在系统相册中，用到微信API：`wx.saveImageToPhotosAlbum(OBJECT)`，需要用户授权，如果用户曾经拒绝过授权，则需要重新唤起授权
    ```js
    function downloadImage(imageUrl) {  
        wx.saveImageToPhotosAlbum({
            // 保存图片到相册
            ...
        })
    }

    onSavePicClick: function(){
        var downloadUrl = this.data.imageUrl
        if (!wx.saveImageToPhotosAlbum) {
            wx.showModal({
                title: '提示',
                content: '当前微信版本过低，无法使用该功能，请升级到最新微信版本后重试。'
            })
            return
        }
        // 可以通过 wx.getSetting 先查询一下用户是否授权了 "scope.writePhotosAlbum" 这个 scope
        wx.getSetting({
            success(res) {
                if (!res.authSetting['scope.writePhotosAlbum']) {  
                    console.log("1-没有授权《保存图片》权限")
                    // 接口调用询问
                    wx.authorize({  
                        scope: 'scope.writePhotosAlbum',  
                        success(){
                            console.log("2-授权《保存图片》权限成功")
                            util.downloadImage(downloadUrl)
                        },  
                        fail(){  
                            console.log("2-授权《保存图片》权限失败")
                            // 打开设置页面
                            wx.openSetting({
                                success: function(data) {
                                    console.log("openSetting: success")  
                                },
                                fail: function(data) {  
                                    console.log("openSetting: fail")
                                }
                            })
                        }
                    })
                } else {  
                    console.log("1-已经授权《保存图片》权限")
                    util.downloadImage(downloadUrl)
                }
            },
            fail(res) {
                console.log("getSetting: success")
            }
        })
    }
    ```

### 坑位
1. 监听onPageScroll事件，动态改变data中的值时，需要加上this.$apply()和async异步触发小程序的响应机制。否则data中的值的改变无法响应。（原因不明）
2. 小程序似乎有保留字。
    引入本地icon图片时，本地的图片名字不能叫arrow.png，否则引不上
    使用css3写keyframes动画时，如果名字为fadeInout，则动画无效
3. 小程序中，复用的页面及组件，并不是单独的实例。
    所以，同一页面，如果复用两次及以上，会出问题。上一个页面data中的数据会被后一个页面的数据替换,并且点击返回按钮后也不会恢复。
    目前的解决方案：页面栈中getCurrentPages()会保留着正确的数据，可以在onshow时从页面栈中找回
4. 对于视图中需要用到的数据，应该事先在data中定义一下，哪怕此时没有数据，也应该定义一个空值
5. 微信官方api，setStorage、getStorage实现有问题，一定概率报错，调用越频繁，报错越多。

    解决：添加try catch，拦截报错，递归调用。封装setStorage、getStorage方法，替代原生api。
    ```javascript
    function setStorage(key, value) {
        try {
            wx.setStorageSync(key, value)
        } catch (e) {
            setStorage(key, value)
        }
    }

    function getStorage(key) {
        try {
            wx.getStorageSync(key)
        } catch (e) {
            getStorage(key)
        }
    }
    ```
6. 在组件中使用`createSelectorQuery()`

    在组件中直接使用`createSelectorQuery()`获取dom节点是无法生效的，正确的姿势是`wx.createSelectorQuery().in(this)`
7. **canvas**
    1. 正常在 page 里如下使用是一点没问题没有的，但是在组件里的话，画不了，就是显示不出来。
    ```
    <canvas class="canvas" canvas-id="canvas"></canvas>
    // 这里的 canvas 就是之前在 wxml 里的 canvas-id
    let ctx = wx.createCanvasContext('canvas')
    ```
    > 【小程序api文档】创建 canvas 绘图上下文（指定 canvasId）。在自定义组件下，第二个参数传入组件实例this，以操作组件内 `<canvas/>` 组件Tip: 需要指定 canvasId，该绘图上下文只作用于对应的 `<canvas/>`

    所以，代码改成如下即可
    ```js
    let ctx = wx.createCanvasContext('canvas', this)
    ```

    2. 通过`wx.getImageInfo`无法将图片链接存到本地使用，开发工具上显示正常，真机就不显示，然后真机打开调试模式又显示正常。

        这里有个坑，小程序对图片链接要求是比较严格的。
        * 图片地址必须是`https`，`http`的不行
        * 链接的域名必须在服务器合法域名和业务域名中（后台配置）
    3. 通过`ctx.arc() 和 ctx.clip()`将图片裁剪成圆形，存在平台兼容问题，ios没毛病，安卓会有很明显的锯齿感。暂无解决方法，目前是将画布大小放大，可以解决。缺点就是：相应的画布渲染时间延长，影响用户体验。
    4. 渲染文字的时候，安卓和iOS存在差异，安卓不解析换行符，而苹果机上解析，导致错乱换行。解决：可以用正则将文案中换行符替换成空格即可。
    8. 1rpx边框问题  
        `1rpx border` ios真机显示不全问题，可能是微信浏览器的渲染机制问题。目前使用`transform: rotateZ(360deg)`可解决。没必要深究。  
        [小程序踩坑日记——(一)1rpx border ios真机显示不全问题](https://www.jianshu.com/p/ac83a405e3af)