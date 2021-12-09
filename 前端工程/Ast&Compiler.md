## AST 与前端编译

### 一. AST简介
抽象语法树（Abstract Syntax Tree）  

![抽象语法树](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/Ast%26Compiler/01.png?x-oss-process=image/resize,w_500)

#### 1. 基本结构
一段代码转换成的抽象语法树是一个对象，改对象会有一个顶级的 type 属性 `Program`；第二个属性是一个数组。  
`body` 数组中存放的每一项都是一个对象，里面包含了所有对于该语句的描述信息
```
type:           描述该语句的类型 --> 变量声明的语句
kind:           变量声明的关键字 --> var
declaration:    声明内容的数组，里面的每一项也是一个对象
    type:   描述该语句的类型
    id:     描述变量名称的对象
        type: 定义
        name: 变量的名字
    init:   初始化变量值的对象
        type:   类型
        value:  值 "is Tree" 不带引号
        row:    "\"is Tree\"" 带引号
```

#### 2. 有什么用
* 语法检查、代码风格检查、格式化代码、语法高亮、错误提示、自动补全
* 代码混淆压缩
* 优化变更代码、改变代码结构等

例如：`function getUserInfo() {}` 更名为 `function a() {}`  
例如：在 `webpack` 中代码编译完成后 `require('a') --> __webpack__require__("*/**/a.js")` 
例如：Taro、uni-app为什么可以一套代码运行多端

### 二. 词法分析和语法分析
`JavaScript` 是解释型语言，一般通过 词法分析 -> 语法分析 -> 语法树，就可以开始解释执行了。  
1. 词法分析（扫描）  
    将字符流转换为记号流（tokens），它会读取代码然后按照一定规则合成一个个标识
    ```js
    // var a = 2
    ;[
        { type: 'Keywork', value: 'var' },
        { type: 'Identifier', value: 'a' },
        { type: 'Punctuator', value: '=' },
        { type: 'Numeric', value: '2' },
    ]
    ```

2. 语法分析（解析器）  
    将词法分析出来的数组转换成树形结构，同时验证语法。语法如果有错的话，抛出语法错误。
    ```js
    {
        ...
        "type": "VariableDeclarator",
        "id": {
            "type": "Identifier",
            "name": "a"
        },
        ...
    }
    ```

### 三. 初步实践：修改函数名
![修改函数名](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/Ast%26Compiler/02.png?x-oss-process=image/resize,w_500)

```js
const esprima = require('esprima')
const estraverse = require('estraverse')
const escodegen = require('escodegen')

const code = `function getUser() {}`
// 生成 AST
const ast = esprima.parseScript(code)
// traverse 方法中有进入和离开两个钩子函数
estraverse.traverse(ast, {
  enter(node) {
    console.log('enter -> node.type', node.type)
    if (node.type === 'Identifier') {
      node.name = 'a'
    }
  },
  leave(node) {
    console.log('leave -> node.type', node.type)
  },
})

const result = escodegen.generate(ast)
console.log(result)

// AST 遍历的流程是深度优先
// enter -> node.type Program
// enter -> node.type FunctionDeclaration
// enter -> node.type Identifier
// leave -> node.type Identifier
// enter -> node.type BlockStatement
// leave -> node.type BlockStatement
// leave -> node.type FunctionDeclaration
// leave -> node.type Program
// 函数名已被修改: function a() {}
```

### 四. 进阶实践：编写一个babel插件
箭头函数转换普通函数
```js
const babel = require('@babel/core')

const code = `const fn = (a, b) => a + b`
const r = babel.transform(code, {
  plugins: ['@babel/plugin-transform-arrow-functions'],
})
console.log(r.code)
// const fn = function fn(a, b) { return a + b; };
```

#### 1. 分析 AST 结构
![分析AST结构](http://img.vanilla.ink/me/webproject/FE-Summary/Engineering/Ast%26Compiler/03.jpeg?x-oss-process=image/resize,w_500)  

1. 箭头函数表达式（`ArrowFunctionExpression`）转换为函数表达式（`FunctionExpression`）
2. 把二进制表达式（`BinaryExpression`）放到一个代码块（`BlockStatement`）中

#### 2. 修改 AST 结构
使用 `@babel/types`：
1. 判断这个节点是不是这个节点（ArrowFunctionExpression 下面的 path.node 是不是一个 ArrowFunctionExpression）
2. 生成对应的表达式

```js
// https://babel.dev/docs/en/babel-types#functionexpression
t.functionExpression(id, params, body, generator, async);

t.blockStatement(body, directives);
```
```js
const babel = require('@babel/core')
const t = require('@babel/types')
const code = `const fn = (a, b) => a + b`
// const code = `const fn = (a, b) => { return a + b }` 需要特殊处理的情况

const arrowFnPlugin = {
  // 访问者模式
  visitor: {
    // 当访问到某个路径的时候进行匹配
    ArrowFunctionExpression(path) {
      // 拿到当前节点
      const node = path.node
      const params = node.params
      let body = node.body
      // 判断是不是 blockStatement，不是的话需要进一步包装
      if (!t.isBlockStatement(body)) {
        // 将原二进制表达式转为 returnStatement
        const returnStatement = t.returnStatement(body)
        // 将 returnStatement 包装为代码块
        body = t.blockStatement([returnStatement])
      }
      // 生成函数表达式
      const functionExpression = t.functionExpression(null, params, body)
      // 替换原节点函数
      path.replaceWith(functionExpression)
    }
  }
}

const r = babel.transform(code, {
  plugins: [arrowFnPlugin],
})
console.log(r.code)
// const fn = function (a, b) { return a + b; };
```

### 五. 一个超小型编译器
[simple-compiler =>>](https://github.com/PercyMo/simple-compiler)

### 六. 一个超小型解释器
TODO: canjs
[canjs](https://github.com/jrainlau/canjs)

### 七. 引用
[the-super-tiny-compiler](https://github.com/jamiebuilds/the-super-tiny-compiler)  

[一文助你搞懂 AST](https://mp.weixin.qq.com/s/-pvoF4vd9jaUEdj0w2zOzA)  

[前端零基础编译原理科普](https://mp.weixin.qq.com/s/Fj6dKyO22-fDqHMndFW5HQ)  