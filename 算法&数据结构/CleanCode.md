## 代码整洁之道

### 一. 数组操作
**熟练掌握 [数组的常用方法 =>>](../JavaScript/Array.md)，有助于写出简洁高效的代码！这极其重要！**

1. **使用 forEach 方法遍历数组，不形成新数组**
    ```js
    // bad
    for (let i = 0; i < arr.length; i++) {
        // todo
        arr[i].key = xx
    }

    // good
    arr.forEach(item => {
        // todo
        item.key = xx
    })
    ```

2. **使用 filter 方法过滤原数组，形成新数组**
    ```js
    let arr = [1, 3, 5, 7]

    // bad
    let newArr = []
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] > 4) {
            newArr.push(arr[i])
        }
    }

    // good
    let newArr = arr.filter(n => n > 4) // [5, 7]
    ```

3. **使用 map 对数组中的所有元素批量处理，形成新数组**
    ```js
    // bad
    let arr = [1, 3, 5, 7],
        newArr = []
    for (let i = 0; i < arr.length; i++) {
        newArr.push(arr[i] + 1)
    }

    // good
    arr.map(n => n + 1) // [2, 4, 6, 8]
    ```

### 二. 其它
1. **通过条件判断给变量赋值布尔值**
    ```js
    // bad
    if (a === 'a') {
        b = true
    } else {
        b = false
    }

    // good
    b = a === 'a'
    ```

2. **在 if 中判断数组长度不为 0**
    ```js
    // bad
    if (arr.length !== 0) {
        // todo
    }

    // good
    if (arr.length) {
        // todo
    }
    ```
    同理，在 if 中判断数组长度为 0
    ```js
    // bad
    if (arr.length === 0) {
        // todo
    }

    // good
    if (!arr.length) {
        // todo
    }
    ```

3. **简单的判断使用三元表达式**
    ```js
    // bad
    if (a === 'a') {
        b = a
    } else {
        b = c
    }

    // good
    b = a === 'a' ? a : c
    ```

4. **使用 Object.keys/Object.values 快速获取对象 键名/键值**
    ```js
    let obj = {
        a: 1,
        b: 2
    }

    // bad
    let keys = []
    for (let key in obj) {
        keys.push(key)
    }

    // good
    let keys = Object.keys(obj) // ['a', 'b']
    ```
    同理，获取对象键值
    ```js
    let obj = {
        a: 1,
        b: 2
    }

    // bad
    let values = []
    for (let key in obj) {
        values.push(obj[key])
    }

    // good
    let values = Object.values(obj) // [1, 2]
    ```

5. **对象解构赋值/重命名/默认值**
    ```js
    // bad
    setForm (person) {
        this.name = person.aaa_bbb_ccc_ddd
        this.age = person.eee_fff_ggg
        this.gender = person.gender
    }

    // good
    setForm ({aaa_bbb_ccc_ddd, eee_fff_ggg, gender}) {
        this.name = aaa_bbb_ccc_ddd
        this.age = eee_fff_ggg
        this.gender = gender
    }

    // best
    // 有时候后端接口返回的字段键名特别长，或者不是小驼峰，可以重命名一下
    setForm ({
        aaa_bbb_ccc_ddd: name,
        eee_fff_ggg: age,
        gender = '男', // 解构时设置默认值
    }) {
        this.name = name
        this.age = age
        this.gender = gender
    }
    ```

6. **|| 短路符设置默认值**
    ```js
    let person = {
        name: '张三',
        age: 38
    }
    let name = person.name || '佚名'
    ```

7. **&& 短路符判断依赖的键是否存在防止报错'xx of undefined'**
    ```js
    let person = {
        name: '张三',
        children: {
            name: '张小三'
        }
    }
    let childrenName = person.children && person.children.name
    ```

8. **字符串拼接使用 \`${}\` 字符串模板**
    ```js
    let person = {
        name: '张三',
        age: 38
    }

    // bad
    function sayHi(person) {
        console.log('大家好，我叫' + person.name + '，我今年' + person.age + '了')
    }

    // good
    function sayHi(person) {
        console.log(`大家好，我叫${person.name}，我今年${person.age}了`)
    }

    // best
    function sayHi({name, age}) {
        console.log(`大家好，我叫${name}，我今年${age}了`)
    }
    ```

9. **使用箭头函数**
    ```js
    let arr = [18, 19, 20, 21, 22]
    // bad
    function findStudentByAge(arr, age) {
        return arr.filter(function (num) {
            return num === age
        })
    }

    // good
    let findStudentByAge = (arr, age) => arr.filter(num => num === age)
    ```

10. **函数参数校验**
    ```js
    // bad
    let findStudentByAge = (arr, age) => {
        if (!age) throw new Error('参数不能为空')
        return arr.filter(num => num === age)
    }

    // good
    let checkoutType = () => {
        throw new Error('参数不能为空')
    }
    let findStudentByAge = (arr, age = checkoutType()) => {
        return arr.filter(num => num === age)
    }
    ```