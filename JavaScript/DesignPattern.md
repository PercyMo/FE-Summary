### 设计模式

#### 1. 发布订阅模式
```js
var pubsub = {};
(function (q) {
    var topics = {},    // 回调函数存放的数组
        subUid = -1;
    
    // 订阅方法
    q.subscrib = function (topic, func) {
        topics[topic] = topics[topic] || [];

        var token = (++subUid).toString();
        topics[topic].push({
            token: token,
            func: func
        });
        return token;
    };

    // 发布方法
    q.publish = function (topic, args) {
        if (!topics[topic]) {
            return false;
        }

        setTimeout(function () {
            var subscribers = topics[topic],
                len = subscribers ? subscribers.length : 0;
            
            while (len--) {
                subscribers[len].func(topic, args);
            }
        }, 0);

        return true;
    };

    // 退订方法
    q.unsubscripe = function (token) {
        for (let m in topics) {
            if (topics[m]) {
                for (let i = 0, len = topics[m].length; i < len; i++) {
                    if (topics[m][i].token === token) {
                        topics[m].splice(i, 1);
                        return token;
                    }
                }
            }
        }
        return false;
    };

}(pubsub));
```

#### 引用
[深入理解JavaScript系列：设计模式](https://www.cnblogs.com/TomXu/)