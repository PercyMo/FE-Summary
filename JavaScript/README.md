### 《JavaScript》学习笔记
> 基于《JavaScript高级程序设计》第三版目录结构

#### 第20章 JSON
1. JSON可以表示三种类型的值：简单值，对象，数组

2. JavaScript字符串和JSON字符串最大的区别在于，JSON字符串必须使用双引号（单引号会导致语法错误）。
    ```json
    "Hello World!"
    ```

3. 于JavaScript对象字面量相比，JSON有三个不一样的地方。
    * 没有声明变量（JSON中没有变量的概念）
    * 末尾没有分号
    * JSON所有对象的属性必须加**双引号**
    ```json
    // JavaScript
    var obj = {
        name: 'Nicholas',
        age: 29
    }

    // JSON
    {
        "name": "Nicholas",
        "age": 29
    }
    ```

4. JSON对象有两个方法：JSON.stringify()和JSON.parse()。分别用于将JSON对象序列化为JSON字符串，和把JSON字符串解析为原生JavaScript值。
    * 值为`undefined`的任何属性都会被跳过