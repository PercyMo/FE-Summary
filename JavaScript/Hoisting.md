### 变量提升和函数提升详解

#### 1. 变量提升   
通常JS引擎会在正式执行之前先进行一次预编译，在这个过程中，首先将变量声明及函数声明提升至当前作用域的顶端，然后进行接下来的处理。   
例子：
```js
var foo = 3;
function hoistVariable() {
    var foo = foo || 5;
    console.log(foo); // 5
}
hoistVariable();
```
`foo || 5`这个表达式的结果是5而不是3，虽然外层作用域有个foo变量，但函数内是不会去引用的，因为预编译之后的代码逻辑是这样的：
```js
var foo = 3;
// 预编译之后
function hoistVariable() {
    var foo;
    foo = foo || 5;
    console.log(foo); // 5
}
hoistVariable();
```
#### 2. 函数提升
```js
function hoistFunction() {
    foo(); // 2
    var foo = function() {
        console.log(1);
    };
    foo(); // 1
    function foo() {
        console.log(2);
    }
    foo(); // 1
}
hoistFunction();
```
运行后我们会发现，输出的结果依次是2 1 1   
因为`JavaScript`中的函数是一等公民，函数声明的优先级最高，会被提升至当前作用域最顶端，所以第一次调用时实际执行了下面定义的函数声明，然后第二次调用时，由于前面的函数表达式与之前的函数声明同名，故将其覆盖，以后的调用也将会打印同样的结果。上面的过程经过预编译之后，代码逻辑如下：
```js
// 预编译之后
function hoistFunction() {
    var foo;
    foo = function foo() {
        console.log(2);
    }
    foo(); // 2
    foo = function() {
        console.log(1);
    };
    foo(); // 1
    foo(); // 1
}
hoistFunction();
```
#### 3. 最佳实践
理解变量提升和函数提升可以使我们更了解这门语言，更好地驾驭它，但是在开发中，我们不应该使用这些技巧，而是要规范我们的代码，做到可读性和可维护性。

具体的做法是：无论变量还是函数，都必须先声明后使用。