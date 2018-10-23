### 原型对象与原型链、继承详解

#### 1. 进程与线程

#### 1. **原型对象与原型链、继承（_重点内容_）**
1. prototype和__proto__的区别

    <img src="https://note.youdao.com/yws/public/resource/0c7e34c500438947463771cc1073655a/xmlnote/WEBRESOURCE247de9dd6ed3d119b222d9620aac2337/899" width="500" />

    ```js
    var a = {};
    a.prototype     // undefined
    a.__proto__     // Object {}

    var b = function() {};
    b.prototype     // b {}
    ```
2. __proto__属性指向谁

    <img src="https://note.youdao.com/yws/public/resource/0c7e34c500438947463771cc1073655a/xmlnote/WEBRESOURCEf94f0664c6cc8dec86029683129a504c/901" width="500" />

    ```js
    // 1. 字面量方式
    var a = {};
    a.__proto__                                 // Object {}
    a.__Proto__ === a.constructor.prototype;    // true

    // 2. 构造器方式
    var A = function() {};
    var a = new A();
    a.__proto__                                 // A {}
    a.__proto__ === a.constructor.prototype;    // true

    // 3. Object.create()方式
    var a1 = {a: 1};
    var a2 = Object.creat(a1);
    a2.__proto__;                               // Object {a: 1}
    a2.__proto__ === a.constructor.prototype    // false（此处即为图1中的例外情况）
    ```
3. 什么是原型链

    <img src="https://note.youdao.com/yws/public/resource/0c7e34c500438947463771cc1073655a/xmlnote/WEBRESOURCEb5e2c35f3974cb59743a6b86f2e23410/897" width="500" />

    ```js
    var A = function() {};
    var a = new A();
    a.__proto__;                                // A {} (即构造器function A 的原型对象)
    a.__proto__.__proto__;                      // Object {} (即构造器function Object 的原型对象)
    a.__proto__.__proto__.__proto__;            // null
    ```
    
    <img src="./images/03.png" width="500" />

    图中由相互关联的原型组成的链状结构就是原型链，也就是蓝色的这条线。


#### 2. 确定原型和实例的关系
* 通过`instanceof`操作符，用来测试实例与原型链中出现过的构造函数，结果就会返回`true`
    ```js
    instance instanceof Object;         // true
    instance instanceof SuperType;      // true
    ```
* 使用`isPrototypeOf()`方法。只要是原型链中出现过的原型，都可以说是该原型链所派生出的实例的原型，`isPrototypeOf()`方法会返回`true`
    ```js
    Object.prototype.isPrototypeOf(instance);       // true
    SuperType.prototype.isPrototypeOf(instance);    // true
    ```
#### 3. 谨慎定义方法

通过原型链实现继承时，不能使用对象字面量创建原型方法，因为这会重写原型链。
```js
// SubType 继承了 SuperType
SubType.prototype = new SuperType();
// 使用对象字面量添加新方法，会导致上一行代码无效
SubType.prototype = {
    getSubValue: function() {
        return this.subproperty
    },
    someOtherMethod: function() {
        return false;
    }
}
```