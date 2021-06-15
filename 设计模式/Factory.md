## 工厂模式
什么情况下使用工厂模式  
* 对象的构建十分复杂
* 需要依赖具体环境创建不同的实例
* 处理大量具有相同属性的小对象

### 一. 简单工厂
#### 1. 介绍
1. 定义  
    定义一个创建对象的类，由这个类来封装实例化对象的行为

2. 角色  
    * Product：抽象产品类
    * ConcreteProduct：具体产品类
    * SimpleFactory：简单工厂类

3. 适用场景  
    * 工厂类负责创建的对象较少

4. 优点  
    * 客户类与具体类解耦
    * 客户类不需要知道所有子类的细节

5. 缺点  
    * 工厂类职责过重
    * 增加工厂类增加了系统复杂度
    * 系统扩展困难（会修改工厂逻辑）

#### 2. 实现
```js
class SukuziCar {
  constructor(color) {
    this.color = color;
    this.brand = 'Sukuzi';
  }
}

class HondaCar {
  constructor(color) {
    this.color = color;
    this.brand = 'Honda';
  }
}

class BMWCar {
  constructor(color) {
    this.color = color;
    this.brand = 'BMW';
  }
}

class CarFactory {
  constructor() {}
  create(brand, color) {
    switch (brand) {
      case 'Sukuzi':
        return new SukuziCar(color);
      case 'Honda':
        return new HondaCar(color);
      case 'BMW':
        return new BMWCar(color);
      default:
        break;
    }
  }
}

const carFactory = new CarFactory()
const car1 = carFactory.create('Sukuzi', 'brown');
const car2 = carFactory.create('Honda', 'grey');
const car3 = carFactory.create('BMW', 'red');
```

### 二. 工厂方法
#### 1. 介绍
1. 定义  
    定义一个创建对象的类，由子类决定要实例化的类。**将产品类的实例化操作延迟到工厂子类中完成。**

2. 角色  
    * Product：抽象产品类
    * ConcreteProduct：具体产品类
    * Factory：抽象工厂类
    * ConcreteFactory：具体工厂类

3. 优点  
    * 添加新产品时只要添加一个工厂和具体产品（符合开闭原则）

4. 缺点  
    * 类个数成倍增加（增加一个产品会增加具体类和实现工厂）

#### 2. 实现
```js
class SukuziCar {
  constructor(color) {
    this.color = color;
    this.brand = 'Sukuzi';
  }
}

class HondaCar {
  constructor(color) {
    this.color = color;
    this.brand = 'Honda';
  }
}

class BMWCar {
  constructor(color) {
    this.color = color;
    this.brand = 'BMW';
  }
}

class CarFactory {
  constructor() {}
  sell(brand, color) {
    const car = this.create(brand, color);
    // do something
    return car;
  }
  create() {
    throw new Error('父类是抽象类不能直接调用，需要子类重写该方法');
  }
}

class SukuziFactory extends CarFactory {
  constructor() {
    super();
  }
  create(color) {
    return new SukuziCar(color);
  }
}

const sukuziFactory = new SukuziFactory();
const car = sukuziFactory.create('brown');
```

### 三. 抽象工厂
#### 1. 介绍
1. 定义  
    为创建一组相关或相互依赖的对象提供一个接口，而且无需指定它们的具体类。**一个工厂可以提供多个产品对象，而不是单一的产品对象。**

2. 角色  
    * Product：抽象产品类
    * ConcreteProduct：具体产品类
    * Factory：抽象工厂类
    * ConcreteFactory：具体工厂类

3. 适用场景  
    * 无需关系对象创建过程
    * 系统中有多于一个产品族，而每次只使用其中一个产品族
    * 系统提供一个产品类的库，所有的产品以同样的接口出现，从而使客户端不依赖于具体实现

4. 优点  
    * 增加新的产品族很容易

5. 缺点  
    * 增加新的产品结构很麻烦（违背开闭原则）

#### 2. 实现
```js
class BMW {}

class BMW730Car extends BMW {
  constructor(color) {
    this.color = color;
    this.brand = 'BMW730';
  }
}

class BMW840Car extends BMW {
  constructor(color) {
    this.color = color;
    this.brand = 'BMW840';
  }
}

class CarFactory {
  constructor() {}
  sell(brand, color) {
    const car = this.create(brand, color);
    // do something
    return car;
  }
}

class BMWFactory extends CarFactory {
  constructor() {
    super();
  }
  create730(color) {
    return new BMW730Car(color);
  }
  create840(color) {
    return new BMW840Car(color);
  }
}

const bmwFactory = new BMWFactory();
const bmw730 = bmwFactory.create730('brown');
const bmw840 = bmwFactory.create840('red');
```