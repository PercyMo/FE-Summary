### 执行上下文和执行上下文栈详解

#### 1. 执行上下文的类型
每当 Javascript 代码在运行的时候，它都是在执行上下文中运行。

* 全局执行上下文
* 函数执行上下文

    每当一个函数被调用时, 都会为该函数创建一个新的上下文。
* Eval 函数执行上下文

#### 2. 执行上下文栈
执行栈，是一种拥有 LIFO（后进先出）数据结构的栈，被用来存储代码运行时创建的所有执行上下文。

为了模拟执行上下文栈的行为，让我们定义执行上下文栈是一个数组：
```js
ECStack = [];
```
当 JavaScript 开始要解释执行代码的时候，最先遇到的就是全局代码，所以初始化的时候首先就会向执行上下文栈压入一个全局执行上下文，我们用 globalContext 表示它，并且只有当整个应用程序结束的时候，ECStack 才会被清空，所以程序结束之前， ECStack 最底部永远有个 globalContext：
```js
ECStack = [
    globalContext
];
```
现在 JavaScript 遇到下面的这段代码了：
```js
function fun3() {
    console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();
```
当执行一个函数的时候，就会创建一个执行上下文，并且压入执行上下文栈，当函数执行完毕的时候，就会将函数的执行上下文从栈中弹出。知道了这样的工作原理，让我们来看看如何处理上面这段代码：
```js
// 伪代码

// fun1()
ECStack.push(<fun1> functionContext);

// fun1中竟然调用了fun2，还要创建fun2的执行上下文
ECStack.push(<fun2> functionContext);

// 擦，fun2还调用了fun3！
ECStack.push(<fun3> functionContext);

// fun3执行完毕
ECStack.pop();

// fun2执行完毕
ECStack.pop();

// fun1执行完毕
ECStack.pop();

// javascript接着执行下面的代码，但是ECStack底层永远有个globalContext
```

#### 3. 怎样创建执行上下文
1. 创建阶段
    在创建阶段会发生三件事：
    1. this 值的决定，即我们所熟知的 This 绑定。
    2. 创建词法环境组件。
    3. 创建变量环境组件。
    所以执行上下文在概念上表示如下：
    ```js
    ExecutionContext = {
        ThisBinding = <this value>,
        LexicalEnvironment = { ... },
        VariableEnvironment = { ... },
    }
    ```
    1. This 绑定：
    
        在全局执行上下文中，this 的值指向全局对象。(在浏览器中，this引用 Window 对象)。

        在函数执行上下文中，this 的值取决于该函数是如何被调用的。如果它被一个引用对象调用，那么 this 会被设置成那个对象，否则 this 的值被设置为全局对象或者 undefined（在严格模式下）。例如：

        ```js
        let foo = {
            baz: function() {
                console.log(this);
            }
        }

        foo.baz();      // 'this' 引用 'foo', 因为 'baz' 被
                        // 对象 'foo' 调用

        let bar = foo.baz;

        bar();          // 'this' 指向全局 window 对象，因为
                        // 没有指定引用对象
        ```
    2. 词法环境
    ...
    3. 变量环境
    ...

2. 执行阶段
...

#### 4. 例子
```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```
```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```
两段代码执行最终都会打印出`local scope`，但是两段代执行上下文栈的变化不一样。

模拟第一段代码：
```js
ECStack.push(<checkscope> functionContext);
ECStack.push(<f> functionContext);
ECStack.pop();
ECStack.pop();
```
模拟第二段代码：
```js
ECStack.push(<checkscope> functionContext);
ECStack.pop();
ECStack.push(<f> functionContext);
ECStack.pop();
```

#### 5. 引用
[JavaScript深入之执行上下文栈](https://github.com/mqyqingfeng/Blog/issues/4)

[ 理解 JavaScript 中的执行上下文和执行栈](https://juejin.im/post/5ba32171f265da0ab719a6d7)