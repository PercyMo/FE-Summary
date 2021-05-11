## 项目部署

### 一. 多环境配置
本司部署时，有测试、预发布、生产三套环境，三套环境的接口地址也不同，通过修改打包命令、模式和环境变量来实现。
```sh
"build": "vue-cli-service build", # 正式环境
"build:test": "vue-cli-service build --mode test", # 测试环境
"build:preprod": "vue-cli-service build --mode preprod", # 预发布环境
```
同时在项目根目录配置相应的环境变量文件
```
.env.preprod
.env.production
.env.test
```
.env.test 为例：
```
NODE_ENV = production
VUE_APP_BASE_API = 'http://test.api.xxx.com/'
```
**环境变量**
`VUE_APP_`开头的变量将通过`webpack.Defineplugin`静态嵌入到客户端侧的代码，可以在应用的代码中访问到：
```js
console.log(precess.env.VUE_APP_BASE_API)
```

