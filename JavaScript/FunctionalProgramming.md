## 函数式编程

### 一. 传统编程范式弊端
#### 1. 面向对象编程
```js
var Flock = function(n) {
  this.seagulls = n;
};

Flock.prototype.conjoin = function(other) {
  this.seagulls += other.seagulls;
  return this;
};

Flock.prototype.breed = function(other) {
  this.seagulls = this.seagulls * other.seagulls;
  return this;
};

var a = new Flock(4);
var b = new Flock(2);
var c = new Flock(0);

var result = a
  .conjoin(c)
  .breed(b) // 在这里 a 已修改为 8
  .conjoin(a.breed(b)) // 计算 a*b 实际 8*2，同时 a 又被修改为 16，最终 16+16 = 32
  .seagulls;

// 预期：(4 + 0) * 2 + 4 * 2 = 16
// 实际：(4 + 0) * 2 =>> 16
//         + 8 * 2 = 32
```
代码内部的可变状态非常难以追踪，而且最终答案是做的。因为 `a` 在运算过程中永久地改变了。  
更换函数式写法：
```js
var conjoin = function(x, y) { return x + y };
var breed = function(x, y) { return x * y };

var a = 4;
var b = 2;
var c = 0;

var result = conjoin(breed(b, conjoin(a, c)), breed(a, b));
// (4 + 0) * 2 + 4 * 2 = 16
```

#### 2. 命令式编程
对已有的人名数组进行一些修改，字符串做一些处理后转对象
```js
['john-reese', 'harold-finch', 'sameen-shaw'] 
// 转换成 
[{name: 'John Reese'}, {name: 'Harold Finch'}, {name: 'Sameen Shaw'}]
```
看下完全面向过程的编程思路：
* 定义临时变量 newArr
* 循环遍历原始数据 arr，控制循环次数 arr.length
* 每个名字首位取出来大写，再拼接上剩余部分
* ...
* 返回结果
```js
var arr = ['john-reese', 'harold-finch', 'sameen-shaw'];
var newArr = [];

for (let i = 0, len = arr.length; i < len; i++) {
  let name = arr[i];
  let names = name.split('-');
  let newName = [];
  for (let j = 0; j < names.length; j++) {
    let nameItem = names[j][0].toUpperCase() + names[j].slice(1);
    newName.push(nameItem);
  }
  newArr.push({ name: newName.join(' ') });
}

return newArr;
```
这么写的问题是，多了一堆中间临时变量。而且整段代码完全混在一起，从头读到尾才知道它具体做了什么。
```js
// 函数式写法
const capitalize = x => x[0].toUpperCase() + x.slice(1).toLowerCase();
const genObj = curry(function(key, value) {
  return {
    [key]: value
  };
});

const capitalizeName = compose(join(' '), map(capitalize), split('-'));
const convert2Obj = compose(genObj('name'), capitalizeName);
const convertName = map(convert2Obj)

convertName(['john-reese', 'harold-finch', 'sameen-shaw']);
```

### 二. 函数式编程特点
流水线，数据可以不断从一个函数的输出流入另一个函数输入，最后再输出结果。  

![函数式编程](http://img.vanilla.ink/me/webproject/FE-Summary/JavaScript/FunctionalProgramming/01.png?x-oss-process=image/resize,w_500)

#### 1. “一等公民”的函数
函数真没什么特殊的，你可以像对待任何其他数据类型一样对待它们——把它们存在数组里，**当做参数传递**，赋值给变量...  
```js
function doSomething () {};

// 一个亲身实践的例子，总感觉函数一定要被执行下，才可以生效
// 实际上，下面两种写法完全等价

setTimeout(() => {
    doSomething();
}, 1000);

setTimeout(doSomething, 1000);
```
```js
const getServerStuff = callback => ajaxCall(json => callback(json));

/**
 * ajaxCall(json => callback(json))
 * 
 * 等价于
 * ajaxCall(callback)
 * 
 * 重构下
 * const getServerStuff = callback => ajaxCall(callback);
 *
 * 继续重构...
 */
const getServerStuff = ajaxCall;
```
添加一些没有实际用处的间接层，除了徒增代码量，提高维护和检索代码的成本，没有任何用处。  
另外，如果一个函数被不必要地包裹起来了，而且发生了改动，那么包裹它的那个函数也要做相应的变更。  
```js
httpGet('/post/2', json => renderPost(json));

// 如果改成可以抛出一个可能出现的 err 异常，那还要把“胶水”函数也改了
httpGet('/post/2', (json, err) => renderPost(json, err));

// 写成一等公民函数的形式，要做的改动将会少得多
httpGet('/post/2', renderPost);
```

#### 2. 声明式编程
大多数时候是在声明我需要做什么，而不是怎么去做。代码接近自然语言，可读性高。  
SQL语句就是声明式。

#### 3. 惰性执行
几乎没有中间变量，从头到尾都在写函数，只在最后才通过调用产生实际结果。

#### 3. 无状态和数据不可变
* **数据不可变**：如果你想修改一个对象，应该创建一个新的对象用来修改，而不是修改已有的对象
* **无状态**：无论何时运行一个函数，都应该像第一次运行一样，给定相同输入，给出相同输出，完全不依赖外部状态的变化

#### 4. 没有副作用
> 副作用是在计算结果的过程中，系统状态的一种变化，或者与外部世界进行的可观察的交互。
* 更改文件系统
* 往数据库插入记录
* 发送一个http请求
* **可变数据**
* 打印/log
* 获取用户输入
* DOM 查询
* 访问系统状态
* ...

并不是说，要禁止使用一切副作用，而是说，要让它们在可控的范围内发生。
```js
// 有副作用
const list = [...];
list.map(item => {
  item.type = 1;
  item.age++;
})

// 无副作用
const list = [...];
const newList = list.map(item => ({ ...item, type: 1, age: item.age + 1 }));
```
保证函数没有副作用，一来保证数据的不可变性，二来能避免很多因为共享状态带来的问题。  
> 传递引用一时爽，代码重构火葬场

#### 5. 纯函数
> 纯函数是这样的一种函数，即相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用。

* **不依赖外部状态（无状态）**：函数的运行结果完全不依赖全局变量，this指针，IO操作等
* **没有副作用（数据不变）**：不修改全局变量，不修改入参

`splice` 产生了可观察的副作用，永久改变了数组。  
在函数式编程中，我们讨厌这种会改变数据的笨函数。我们追求的是那种可靠的，每次都能返回同样结果的函数。  
```js
var xs = [1, 2, 3, 4, 5];

// 纯的
xs.slice(0, 3); // [1, 2, 3]
xs.slice(0, 3); // [1, 2, 3]
xs.slice(0, 3); // [1, 2, 3]

// 不纯的
xs.splice(0, 3); // [1, 2, 3]
xs.splice(0, 3); // [4, 5]
xs.splice(0, 3); // []
```
另一个例子：
```js
// 不纯的：函数的结果取决于 mininum 这个可变变量的值，或者说，它取决于系统状态
var mininum = 21;

var checkAge = function (age) {
  return age >= mininum;
};

// 纯的：自给自足
var checkAge = function (age) {
  var mininum = 21;
  return age >= mininum;
};
```

1. 从数学角度看纯函数
    > 数学中对函数的定义：函数是不同数值之间的特殊关系，每一个输入值返回且只返回一个输出值

    如果输入直接明了输出，函数仅仅是输入到输出的映射而已，所以简单地写一个对象就能“运行”它，使用`[]`代替`()`即可。
    ```js
    var toLowerCase = {'A': 'a', 'B': 'b', 'C': 'c', 'D': 'd', 'E': 'e', 'D': 'd'};

    toLowerCase['C']; // c

    var isPrime = {1: false, 2: true, 3: true, 4: false, 5: true, 6: false};

    isPrime[3]; // true
    ```
    当然了，实际情况中你可能需要进行一些计算而不是手动指定各项值，但这表明了另一种思考函数的方式。  
    这里借用阮一峰的一段描述：  
    > 为什么函数式编程要求函数必须是纯的，不能有副作用？因为它是一种数学运算，原始目的就是求值，不做其他事情，否则就无法满足函数运算法则了。  
    > 总之，在函数式编程中，函数就是一个管道(pipe)。这头进去一个值，那头就会出来一个新的值，没有其他作用。

2. 纯函数优点
    1. **可缓存性**  
        纯函数可以根据输入来做缓存。
        ```js
        const squareNumber = memoize(function(x) { return x * x; });

        squareNumber(4); // 16
        squareNumber(4); // 16 => 从缓存中读取输入值为 4 的结果
        squareNumber(5); // 25
        squareNumber(5); // 25 => 从缓存中读取
        ```
        缓存函数的简单实现：
        ```js
        function memoize(f) {
          const cache = {};
          return function() {
            const key = JSON.stringify(arguments);
            if (!cache[key]) {
              cache[key] = f.apply(f, arguments);
            }
            return cache[key];
          }
        }
        ```

    2. **可移植性/自文档化**  
        * 可移植性：命令式编程中“典型”的方法和过程都深深植根于它们所在的环境中，通过状态、依赖和有效作用达成；纯函数与此相反，它与环境无关，只要我们愿意，可以在任何地方运行它。
        * 自文档化：纯函数的依赖很明确，因此更易于观察和理解——没有偷偷摸摸的小动作。

    3. **可测试性**  
        由于纯函数对于相同的输入永远会返回相同的结果，因此可以轻松断言函数的执行结果，同时可以保证函数的优化不会影响其他代码的执行。这十分符合 **测试驱动开发 TDD** 的思想，这样产生的代码往往更健壮。

    4. **更少的 Bug**  
        不存在指向不明的 this；不存在对全局变量的引用；不存在对参数的修改。

### 三. 流水线的构建
#### 1. 加工站——柯里化
柯里化是将一个多元函数，转换成一个依次调用的**单元函数**。
```js
// fn(a, b, c) -> fn(a)(b)(c)


const add = function(x) {
  return function(y) {
    return x + y;
  };
};

const increment = add(1); // function
increment(10); // 11
```
为什么需要柯里化：因为函数式编程组装流水线，需要保证每个加工站的输出刚好能流向下个工作站的输入，**因此，在流水线上的加工站必须都是单元函数。**

1. **柯里化的应用**  
    ```js
    const replace = curry((a, b, str) => string.replace(a, b));
    const replaceSpaceWith = replace(/\s*/);
    const replaceSpaceWithComma = replaceSpaceWith(',');
    const replaceSpaceWithDash = replaceSpaceWith('-');
    ```
    * 将函数变得单值化，从一个`replace`函数中产生很多新函数，可以在各种场合进行使用
    * 更重要的，单值函数是**函数组合的基础**

#### 2. 流水线——函数组合
```js
const compose = (f, g) => x => f(g(x));

const f = x => x + 1;
const g = x => x * 2;
const fg = compose(f, g) // compose 返回了一个新函数，g -> f 的流水线
fg(1); // 3
```
`compose` 满足结合律
```js
compose(f, compose(g, t)) = compose(compose(f, g), t) = f(g(t(x)))

// Ramda 等库中更高级的 compose
compose(f, g, t) = x => f(g(t(x))) // t -> g -> f
```

1. **函数组合的应用**  
    例子：将数组最后一个元素大写，假设`log`,`head`,`reverse`,`toUpperCase`函数存在
    ```js
    // 流水线：reverse -> head -> toUpperCase -> log

    // 命令式
    log(toUpperCase(head(reverse(arr))));

    // 函数式组合
    const upperLastItem = compose(log, toUpperCase, head, reverse);

    // 借助 Ramda
    const upperLastItem = R.pipe(reverse, head, toUpperCase, log);
    ```

2. **函数组合的好处**  
    * 代码简单且富有可读性
    * 通过不同组合方式，组合出其他常用函数

      ```js
      const lastUpper = compose(toUpperCase, head, reverse);
      const logLastUpper = compose(log, lastUpper);
      ```

### 四. 实践经验
#### 柯里化中把要操作的数据放到最后
```js
const split = curry((x, str) => str.split(x));
const join = curry((x, arr) => arr.join(x));
const replaceSpaceWithComma = compose(join(','), split(' '));
```
如果有些函数没有遵循这个约定，很多库提供了占位符的概念，如 Ramada 的`R.__`。
```js
const split = curry((str, x) => str.split(x));
const replaceSpaceWithComma = compose(join(','), split(R.__, ' '));
```

#### 函数组合中函数要求单输入

#### 函数组合的 Debug
```js
// 借助一个辅助函数
const trace = curry((tip, x) => {
  console.log(tip);
  return x;
});
const lastUpper = compose(toUpperCase, head, trace('after reverse'), reverse);
```

### 五. 实战
现有一套数据
```js
const data = [
  {
    name: 'Peter',
    sex: 'M',
    age: 18,
    grade: 99
  },
  ……
]
```
```js
/**
 * 获取所有年龄小于 18 岁的对象，并返回他们的名称和年龄
 */
const ageUnder18 = R.pipe(
  R.prop('age'),
  R.lt(R.__, 18)
);

const getAgeUnder18 = R.pipe(
  R.filter(ageUnder18),
  R.map(R.pickAll(['name', 'age']))
);


/** 
 * 根据性别查找用户
 */
const getBySex = sex => R.filter(R.propEq('sex', sex));
const getMales = getBySex('M');
const getFemales = getBySex('F');


/** 
 * 更新一个指定名称用户的成绩（不影响原数组）
 */
const updatePropBy = R.curry((byProp, updateProp, match, newValue, obj) => R.when(
    R.propEq(byProp, match),
    R.assoc(updateProp, newValue)
  )(obj)
);
const updatePropByName = updatePropBy('name');
const updateGradeByName = updatePropByName('grade');
const updateUsersGradeByName = R.curry((name, grade, arr) => R.map(updateGradeByName(name, grade), arr));


/** 
 * 取出成绩最高的 10 名，并返回他们的名称和分数
 */
const descendBy = x => R.sort(R.descend(R.prop(x)));
const getGradeTop10 = R.pipe(
  descendBy('grade'),
  R.take(10),
  R.map(R.pickAll(['name', 'grade']))
);
```

### 六. 总结
**优点**：
* 代码间接，开发快速：大量使用函数的组合，复用率高，减少代码重复
* 接近自然语言，易于理解：声明式代码，没有乱七八糟的循环和判断的嵌套
* 易于并发编程：没有副作用，适于在 web worker 中运行
* 更少的出错概率：每个函数都很小，测试简单，使用纯函数，没有副作用

**缺点**：
* 性能：往往会对一个方法进行过度包装，从而产生上下文切换的性能开销。原生`map`要比纯循环语句实现迭代慢 8 倍
* 资源占用：为了实现对象不可变状态，往往会创建新对象

在日常工作中将函数式编程作为一种辅助手段，在条件允许的前提下，借鉴函数式编程中的思路：
* 多使用纯函数减少副作用的影响
* 使用柯里化增加函数适用率
* 使用 Pointfree 编程风格，减少无意义的中间变量，让代码可读性更强
* ...

> **没有最好的，只有最合适的**

### 六. 引用
[简明 JavaScript 函数式编程——入门篇](https://juejin.cn/post/6844903936378273799)  

[Understanding Functional Programming in Javascript — A Complete Guide](https://levelup.gitconnected.com/understanding-functional-programming-in-javascript-a-complete-guide-e85ed13b42c8)

[函数式编程指北](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/)  
