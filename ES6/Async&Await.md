## 示例标题

### 一. 一级标题a

### 二. 一级标题b

#### 1. 二级小标题
1. a
```js
let a = 'hello world'
```

2. b
<img src="./images/EventLoop/01.jpg" width="400" />

### 三. 总结

### 四. 引用
[引用](http://www.baidu.com)


### 待整理
```js
let a = 0
let b = async () => {
  a = a + await 10
  console.log('2', a) // -> '2' 10
}
b()
a++
console.log('1', a) // -> '1' 1
```
对于以上代码你可能会有疑惑，让我来解释下原因

* 首先函数 b 先执行，在执行到 await 10 之前变量 a 还是 0，因为 await 内部实现了 generator ，generator 会保留堆栈中东西，所以这时候 a = 0 被保存了下来
* 因为 await 是异步操作，后来的表达式不返回 Promise 的话，就会包装成 Promise.reslove(返回值)，然后会去执行函数外的同步代码
* 同步代码执行完毕后开始执行异步代码，将保存下来的值拿出来使用，这时候 a = 0 + 10
上述解释中提到了 await 内部实现了 generator，其实 await 就是 generator 加上 Promise的语法糖，且内部实现了自动执行 generator。如果你熟悉 co 的话，其实自己就可以实现这样的语法糖。




另外 await 的例子其实可以转换为（评论中的不同看法）
```js
var a = 0
var b = () => {
  var temp = a;
  Promise.resolve(10)
    .then((r) => {
      a = temp + r;
    })
    .then(() => {
      console.log('2', a)
    })
}
b()
a++
console.log('1', a)
```