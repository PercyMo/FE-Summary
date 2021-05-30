## 建造者模式

### 一. 介绍
#### 1. 定义
将一个复杂对象分解成多个相对简单的部分，然后根据不同需要分别创建它们，最后构建成该复杂对象。

#### 2. 适用场景
* 需要生成的产品对象有复杂的内部结构，这些产品对象通常包含多个成员属性
* 需要生成的产品对象的属性相互依赖，需要指定其生成顺序
* 隔离复杂对象的创建和使用，并使得相同的创建过程可以创建不同的产品
* GitHub - node-sql-query（一个典型示例）

#### 3. 优点
* 建造者模式解耦了组装过程和创建具体部件，使得我们不用关心每个部件是如何组装的。
* 具体建造者相互独立，扩展容易
* 可以精确控制产品创建过程

#### 4. 缺点
* 使用范围受限，要求所创建的产品一般具有较多共同点
* 产品内部变化复杂会导致具体建造者过多

### 二. 实现
```js
class BookBuilder {
  constructor() {
    this.name = ''
    this.author = ''
    this.price = 0
    this.category = ''
  }

  withName(name) {
    this.name = name
    return this
  }

  withAuthor(author) {
    this.author = author
    return this
  }

  withPrice(price) {
    this.price = price
    return this
  }

  withCategory(category) {
    this.category = category
    return this
  }

  build() {
    return {
      name: this.name,
      author: this.author,
      price: this.price,
      category: this.category
    }
  }
}

const book = new BookBuilder()
  .withName("高效能人士的七个习惯")
  .withAuthor('史蒂芬·柯维')
  .withPrice(51)
  .withCategory('励志')
  .build()
```