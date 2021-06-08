## 文本加密
[开源js库：crypto-js](https://github.com/brix/crypto-js)  
`crypto-js`中包含了多种算法，可以根据应用按需引入
```js
// 所有代码都会被打包进项目
import CryptoJS from 'crypto-js'
CryptoJS.MD5("Message")

// 只有aes相关代码会被打包进项目
import MD5 from 'crypto-js/md5'
MD5("Message")
```

### 一. 对称加密算法

### 二. 非对称加密算法
#### AES
高级加密标准（Advanced Encryption Standard），常见的对称加密算法（微信小程序加密传输就是用的这个加密算法）。
```js
import AES from 'crypto-js/aes'
import UTF8 from 'crypto-js/enc-utf8'

const key = UTF8.parse('2q6LwKAuDcppKhye') // 秘钥
const iv = UTF8.parse('0102030405060708') // 偏移量

function Encrypt(word) {
  const srcs = UTF8.parse(word)
  return AES.encrypt(srcs, key, { iv }).toString()
}

Encrypt(JSON.stringify({x: 0.4362745, y: 0.60294116})) // wDjumZGG2kCyrWnYjxY76nFs84uugOa+wNCFozrT5CU=
```

### 三. Hash算法

### 四. Base64编码

