## 函数节流和防抖
TODO: 具体内容待填充...
1. 绑定this指向
2. 正确接收传参

### 一. 函数节流
```js
function throttle(fn, wait) {
    var timer = null
    return function() {
        var self = this
        var args = arguments
        if (!timer) {
            timer = setTimeout(function() {
                timer = null
                fn.apply(self, args)
            }, wait)
        }
    }
}
```

### 二. 函数防抖
```js
function debounce(fn, wait) {
    var timer = null
    return function() {
        var self = this
        var args = arguments
        clearTimeout(timer)
        timer = setTimeout(function() {
            fn.apply(self, args)
        }, wait)
    }
}
```

### 四. 引用
[JavaScript专题之跟着underscore学节流，防抖](https://github.com/mqyqingfeng/Blog/issues/22)