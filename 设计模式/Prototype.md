## 原型模式

### 一. 介绍
#### 1. 定义
原型实例指向创建对象的种类，并且通过拷贝这些原型创建新的对象。

#### 2. 适用场景
* 创建成本比较大的场景；
* 需要动态获取当前的对象运行状态的场景；

#### 3. 优点
* 简化创建对象的过程并提高效率；
* 可动态获取对象运行时的状态；
* 原始对象变化（增加或减少属性）响应克隆对象也会变化

#### 4. 缺点
* 多个实例间修改属性可能互相影响（优点也是缺点），注意深浅拷贝的问题
    ```js
    var person = {
      name: "percy",
      friends: ["zhang", "ma"],
    };
    var person1 = Object.create(person);
    var person2 = Object.create(person);

    person1.name = "li";
    console.log(person2.name); // 基本类型未受影响

    person1.friends.push("wu");
    console.log(person2.friends); // person2 莫名其妙也多了个朋友

    // 可以组合使用构造函数和原型模式解决以上问题
    ```

### 二. 实现
```js
var someCar = {
  name: "凯迪拉克",
  drive: function () {},
};

// 使用 Object.create 创建一个新的车
var anotherCar = Object.create(someCar);
anotherCar.name = "雪佛兰";
```
在 `Object.create` 的第二个参数使用对象字面量传入要初始化的额外属性。
```js
var someCar = {
  name: "凯迪拉克",
  drive: function () {},
};

var anotherCar = Object.create(someCar, {
  id: {
    value: 1,
    enumerable: true, // 默认 writable: false, configurable: false
  },
});
```