## new 运算符详解

### 一. new 的过程都发生了什么
1. 创建一个新对象
2. 使新对象的 `__proto__` 指向构造函数的 `prototype`
3. 使用`apply`调用构造函数，并传入参数
4. 当构造函数执行返回结果为`object`时，返回该`object`，否则返回新对象

### 二. 模拟现实
[new 模拟实现 =>>](../算法&数据结构&手撕代码/JavaScript.md#手写-new)

### 三. 特性
1. 当构造函数有返回值，且返回值类型为对象
    ```js
    function Person (name, age) {
        this.strength = 60;
        this.age = age;

        return {
            name: name,
            habit: 'Games'
        }
    }

    var person = new Person('Kevin', '18');

    console.log(person.name)      // Kevin
    console.log(person.habit)     // Games
    console.log(person.strength)  // undefined
    console.log(person.age)       // undefined
    ```

2. 当构造函数有返回值，且返回值为基本类型
    ```js
    function Person (name, age) {
        this.strength = 60;
        this.age = age;

        return 'handsome boy';
    }

    var person = new Person('Kevin', '18');

    console.log(person.name)        // undefined
    console.log(person.habit)       // undefined
    console.log(person.strength)    // 60
    console.log(person.age)         // 18
    ```

### 四. 顺带提一下Object.create的模拟现实
[深浅拷贝 =>>](../算法&数据结构&手撕代码/JavaScript.md#objectcreate)

### 五. 引用
[JavaScript深入之new的模拟实现](https://github.com/mqyqingfeng/Blog/issues/13)
