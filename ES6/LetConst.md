## let 和 const

### 一. 块级作用域的出现
通过`var`声明的变量存在变量提升的特性。
```js
if (condition) {
    var value = 1;
}
console.log(value);

// 由于变量提升，上面的代码相当于：
var value;
if (condition) {
    value = 1;
}
console.log(value);
// 当 condition 为 false 时，输出 undefined
```
除此之外，在for循环中：
```js
for (var i = 0; i < 10; i++) {
    // ...
}
console.log(i); // 10
// 即便循环已经结束了，我们依然可以访问到 i 的值
```
块级作用域存在于：
* 函数内部
* 块中（字符{ 和 }之间的区域）

### 二. let 和 const
块级声明用于声明在指定的块级作用域外无法访问的变量。`let`和`const`都是块级声明的一种。
1. **不会被提升**  
    ```js
    if (false) {
        let value = 1;
    }
    console.log(value); // Uncaught ReferenceError: value is not defined
    ```

2. **重复声明报错**  
    ```js
    var value = 1;
    let value = 1;  // Uncaught SyntaxError: Identifier 'value' has already been declared
    ```

3. **不绑定全局作用域**  
    当在全局作用域中使用`var`声明时，会创建一个新的全局变量作为全局对象的属性
    ```js
    var value = 1;
    console.log(window.value);  // 1
    ```

    然而`let`和`const`不会
    ```js
    let value = 1;
    console.log(window.value);  // undefined
    ```

4. **let 和 const的区别**  
    `const`用于声明常量，其值一旦被设定不能再被修改，否则会报错。  
    值得一提的是：`const`声明不允许修改绑定，但允许修改值。这意味着当用`const`声明对象时：
    ```js
    const data = {
        value: 1
    };

    // 没有问题
    data.value = 2;
    data.num = 3;

    // 报错
    data = {};  // Uncaught TypeError: Assignment to constant variable.
    ```

### 三. 临时死区
临时死区（Temporal Dead Zone），简写为`TDZ`。  
let 和 const 声明的变量不会被提升到作用域顶端，如果在声明之前访问这些变量，会导致报错：
```js
console.log(typeof value);  // Uncaught ReferenceError: value is not defined
let value = 1;
```

这是因为`JavaScript引擎`在扫描代码发现变量声明时，要么将它们提升到作用域顶部（遇到`var`声明），要么将声明放到`TDZ`中（遇到`let`和`const`声明）。访问`TDZ`中的变量会触发运行时错误。只有执行过变量声明语句，变量才会从`TDZ`中移出，然后方可访问。
```js
var value = 'global';

// 例子1
(function() {
    console.log(value);
    let value = 'local';
}());

// 例子2
{
    console.log(value);
    const value = 'local';
}

// 上面两个例子中，结果并不会打印 'global'，而是报错：Uncaught ReferenceError: value is not defined
// 就是因为TDZ的缘故
```

### 四. 循环中的块级作用域
```js
var funcs = [];
for (var i = 0; i < 3; i++) {
    funcs[i] = function() {
        console.log(i);
    }
}
funcs[0](); // 3
```
`ES6`的`let`为这个问题提供了新的解决方案：
```js
var funcs = [];
for (let i = 0; i < 3; i++) {
    funcs[i] = function() {
        console.log(i);
    }
}
funcs[0](); // 0
```
`let`声明在循环内部的行为是标准中专门定义的，不一定就与`let`的不提升特性有关。  
[ECMAScript 规范第 13.7.4.7 节:](http://www.ecma-international.org/ecma-262/6.0/#sec-for-statement-runtime-semantics-labelledevaluation)  
在`for`循环中使用`let`和`var`，底层会使用不同的处理方式。  
简单来说，就是在`for (let i = 0; i < 3; i++)`中，即圆括号之内建立一个隐藏的作用域，这就可以解释为什么：
```js
for (let i = 0; i < 3; i++) {
    let i = 'abc';
    console.log(i);
}
// abc
// abc
// abc
```
然后**每次迭代循环时都创建一个新变量，并以之前迭代中同名变量的值将其初始化**。这样对于下面这样一段代码
```js
var funcs = [];
for (let i = 0; i < 3; i++) {
    funcs[i] = function() {
        console.log(i);
    };
}
funcs[0](); // 0
```
就相当于：
```js
// 伪代码
(let i = 0) {
    funcs[0] = function() {
        console.log(i);
    };
}

(let i = 1) {
    funcs[1] = function() {
        console.log(i);
    };
}

(let i = 2) {
    funcs[2] = function() {
        console.log(i);
    };
}
```

### 五. 循环中的let和const
```js
var funcs = [], object = {a: 1, b: 1, c: 1};
for (var key in object) {
    funcs.push(function(){
        console.log(key)
    });
}
funcs[0](); // c
```
如果把`var`改成`let`，结果自然会是`a`；  
**如果改成`const`，结果不会报错，而是也会正确打印出`a`**。这是因为在`for in`循环中，每次迭代不会修改已有的绑定，而是会创建一个新的绑定。

### 六. Babel如何编译
```js
let value = 1;

// 编译后
var value = 1;
```

```js
if (false) {
    let value = 1;
}
console.log(value); // Uncaught ReferenceError: value is not defined

// 编译后
if (false) {
    var _value = 1;
}
console.log(value);
```

再看循环中的`let`声明：
```js
var funcs = [];
for (let i = 0; i < 3; i++) {
    funcs[i] = function() {
        console.log(i);
    };
}
funcs[0](); // 0

// 编译后
var funcs = [];

var _loop = function(i) {
    funcs[i] = function() {
        console.log(i);
    };
}
for (var i = 0; i < 3; i++) {
    _loop(i);
}
funcs[0](); // 0
```

### 七. 最佳实践
默认使用`const`，只有当确实需要改变变量的值时才使用`let`。这是因为大部分的变量的值在初始化后不应再改变，而预料之外的变量值的改变是很多`bug`的源头。

### 八. 引用
[ES6 系列之 let 和 const](https://github.com/mqyqingfeng/Blog/issues/82)