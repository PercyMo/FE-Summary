### 架构模式MVC和MVVM

#### 1. MVC
1. 基本模式
    1. View 传送指令到 Controller

        视图层最主要的是监听模型层上的数据改变，当然也包括一些事件的注册或者ajax请求操作(发布事件)。

    2. Controller 完成业务逻辑后，要求 Model 改变状态

        控制器接收用户的操作，最主要是订阅视图层的事件，然后调用模型或视图去完成用户的操作。

    2. Model 将新的数据发送到 View，用户得到反馈

        模型用于封装与应用程序的业务逻辑相关的数据以及对数据处理的方法。模型有对数据直接访问的权利。模型不依赖 “视图” 和 “控制器”, 也就是说，模型它不关心页面如何显示及如何被操作。

    <img src="./images/04.png" width="400" />

    所有通信都是单向的。

2. 代码示例
    1. html部分
    ```html
    <div class="num" id="num"></div>
    <div class="btn" id="increase">+</div>
    <div class="btn" id="decrease">-</div>
    ```

    2. js部分
    ```js
    var app = {};

    // 数据层
    app.Model = function () {
        var val = 0;

        this.add = function (v) {
            if (val < 100) val += v;
        };

        this.sub = function (v) {
            if (val > 0) val -= v;
        };

        this.getValue = function () {
            return val;
        };

        // 观察者模式
        var _this = this,
            views = [];

        this.register = function (view) {
            views.push(view);
        };

        this.notify = function () {
            for (var i = 0, len = views.length; i < len; i++) {
                views[i].render(_this);
            }  
        };
    };

    // 视图层
    app.View = function (controller) {

        var $num = $('#num'),
            $incBtn = $('#increase'),
            $decBtn = $('#decrease');

        this.render = function (model) {
            $num.text(model.getValue() + 'rmb');
        };

        $incBtn.click(controller.increase);
        $decBtn.click(controller.decrease);
    };

    // 控制器
    app.Controller = function () {
        var view = null,
            model = null;

        this.init = function () {
            view = new app.View(this);
            model = new app.Model();

            model.register(view);
            model.notify();
        };

        this.increase = function () {
            model.add(1);
            model.notify();
        };

        this.decrease = function () {
            model.sub(1);
            model.notify();
        };
    };

    // 控制器初始化
    (function() {
        var controller = new app.Controller();
        controller.init();
    })();
    ```

#### 2. MVVM
1. 数据绑定

    主流的MVVM框架实现数据绑定的方式大概分为以下几种：
    * 数据劫持 (Vue)
    * 发布-订阅模式 (Knockout、Backbone)
    * 脏值检查 (Angular)

2. 数据劫持的模拟实现
    ```js
    var data = {
        vision: '1.0',
        name: 'vue',
        arr: [1, 2, 4],
        obj: {
            test: 'asd'
        }
    };

    function observeAllKey(data) {
        if (!data || typeof data !== 'object') {
            return;
        }

        // 使用递归劫持对象属性
        Object.keys(data).forEach((key) => {
            observer(data, key, data[key]);
        });
    };

    function observer(data, key, value) {
        // 监听子属性
        observeAllKey(value);

        Object.defineProperty(data, key, {
            get: function () {
                return value;
            },
            set: function (val) {
                if (val === value) {
                    return;
                }
                console.log(`监听成功：${value} ==> ${val}`);
                value = val;
            }
        });
    };

    observeAllKey(data);
    data.arr[1] = 5;    // 监听成功：2 ==> 5
    ```

#### 3. 引用
[浅析前端开发中的 MVC/MVP/MVVM 模式](https://juejin.im/post/593021272f301e0058273468)