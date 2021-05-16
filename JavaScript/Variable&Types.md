## 变量和类型

### 一. 一级标题a

### 二. 一级标题b

#### 1. 二级小标题

1. **使用`toString`来获取准确的引用类型**
每个引用类型都有`toString`方法，默认情况下，`toString()`方法被每个`Object`对象继承。如果此方法在自定义对象中未被覆盖，`toString()`返回`"[Object type]"`，其中`type`是对象的类型。  
注意，上面提到了如果此方法在自定义对象中未被覆盖，`toString`才会达到预想的效果，事实上，大部分引用类型比如`Array、Date、RegExp`等都重写了`toString`方法。  
我们可以直接调用`Object`原型上未被覆盖的`toString()`，使用`call`来改变`this`指向来达到我们想要的效果。
```js
Object.prototype.toString.call(target)
```

### 三. 数据类型转换
**隐式类型转换**：`Number(value)`  
**显示类型转换**：`1 == null`  
**在js中只有三种类型的转换**：  
* 转换为Boolean类型：`Boolean()`
* 转换为String类型：`String()`/`toString()`
* 转换为Number类型：`Number()`/`parseInt()`/`parseFloat()`

#### 1. 转为boolean
1. 显示：`Boolean()`  
2. 隐式：通常在逻辑判断或者有逻辑运算符时被触发（&& || ！）  
    ```js
    Boolean(2)    // 显示
    if (2) {}     // 逻辑判断
    !!2           // 逻辑运算符
    2 || 'hello'  // 逻辑运算符

    // 逻辑运算符是在内部做了 boolean 类型转换，但实际上返回的是原始操作数的值，即使他们都不是 boolean 类型
    let x = 'hello' && 123  // x === 123
    ```

boolean 类型转换只会有true或者false两种结果。  
**除了`0/NaN/空字符串/null/undefined`五个值是false，其余都是true**  

#### 2. 转为string
1. 显示：`String()/toString()`  
    ```js
    [1, 2, 3].toString()    // "1,2,3"
    String({})              // "[object Object]"
    ```
2. 隐式：通常在有 + 运算符并且有一个操作数是string类型时被触发  
    ```js
    1 + '123'   // "1123"
    1 + {}      // "1[object Object]"
    ```
    1. 有两边，一边是字符串，则会变成字符串拼接
    2. 有两边，一边是对象
#### 3. 转为number
1. 显示：`Number()`  
    * 字符串转换为数字：空字符串变为0，如果出现任何一个非有效数字字符，结果都是NaN  
        ```js
        Number('')      // 0
        Number('10px')  // NaN
        Number('10')    // 10
        ```
    * 布尔值转换为数字  
        ```js
        Number(true)    // 1
        Number(false)   // 0
        ```
    * null和undefined转换为数字  
        ```js
        Number(null)        // 0
        Number(undefined)   // NaN
        ```
    * Symbol无法转换为数字，会报错  
        ```js
        Number(Symbol()) // Uncaught TypeError: Cannot convert a Symbol value to a number
        ```
    * BigInt 去除“n”  
        ```js
        Number(12312412321312312n) // 12312412321312312
        ```
    * 对象转换为数字，会按照下面的步骤去执行  
        1. 先调用对象的 `Symbol.toPrimitive` 方法，如果不存在这个方法
        2. 再调用 `valueOf` 获取原始值，如果获取的值不是原始值
        3. 再调用 `toString` 把其变为字符串
        4. 最后把字符串基于 `Number()` 方法转为数字
        ```js
        let obj = {
            name: 'xxx'
        }
        console.log(obj - 10) // 先把obj隐式转换为数字，再进行运算
        obj[Symbol.primitive]       // undefined
        obj.valueOf()               // {name: "xxx"}
        obj.toString()              // "[object Object]"
        Number("[object Object]")   // NaN
        Nan - 10                    // NaN
        ```
2. 隐式：下面多种情况都会触发隐式类型转换  
    * 比较操作（>,<,<=,>=）
    * 按位操作（| & ^ ~）
    * 算术操作（- + * / %），*注意：*当+操作存在任意的操作数是string类型时，不会触发 number 类型的隐式转换
    * 一元 + 操作

#### 4. 操作符==两边的隐式转换规则
如果两边数据类型不同，需要先转换为相同类型，然后再进行比较
1. **对象==字符串**  
    ```js
    [1, 2, 3] == '1, 2, 3'      // true
    [1, 2, 3][Symbol.primitive] // undefined
    [1, 2, 3].valueOf()         // [1, 2, 3]
    [1, 2, 3].toString()        // "1,2,3"
    ```
2. **null/undefined**  
    ```js
    null == undefined   // true
    null === undefined  // false
    ```
3. **对象==对象**  
    ```js
    {} == {}   // false 内存地址不同
    ```
4. **NaN**  
    ```js
    NaN == NaN  // false
    ```
除了以上情况，只要两边不一致，剩下的都是转换为数字，然后再进行比较

#### 5. 特殊情况
```js
{} + [] === 0   // true
[] + {} === 0   // false

/**
  * 对于编辑器而言，代码块不会反悔任何的值
  * 接着 +[] 就变成了一个强制转 number 的过程
  */
```

#### 6. 面试题
```js
let a = 100 + true + 21.2 + null + undefined + 'Tencent' + [] + null + 9 + false;
// 'NaNTencentnull9false'
```

### 四. 引用
[通过大厂面试题研究JavaScript数据类型转换](https://mp.weixin.qq.com/s/_9zypdZiuC7evMmR50DZdQ)