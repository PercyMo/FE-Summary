## Nuxt.js 实践
### 一. 基础应用与配置
#### 1. context
> context 是 Nuxt 额外提供的对象，在asyncData、plugins、middlewares、modules和store/nuxtServerlint等特殊的 Nuxt 生命周期区域中都会使用到 context。
1. **app**  
    类似于`vue`中的`this`，全局属性和方法都会挂载到上面。Nuxt提供的生命周期都是运行在服务端，会先于`vue`实例创建，因此在这些生命周期中无法通过`this`获取实例上的方法和属性。使用`app`可以来弥补这点，一般我们会把全局的方法同时注入到`this`和`app`中，在服务端的生命周期中使用`app`去访问，而在客户端中使用`this`，保证方法的共用。  
    假设`$axios`已被同时注入，一般主要数据通过`asyncData`（该生命舟曲发起请求，将获取到的数据交给服务端拼接成`html`返回）去提前请求做服务端渲染，而次要数据通过客户端的`mounted`去请求。  
    ```js
    export default {
      async asyncData({ app }) {
        // 列表数据
        const list = await app.$axios.getIndexList({
          params: {
            pageNum: 1,
            pageSize: 20,
          },
        }).then(res => res.success ? res.data : [])
        return {
          list,
        }
      },
      data() {
        return {
          list: [],
          categories: [],
        }
      },
      async mounted() {
        // 分类
        const res = await this.$axios.getCategories()
        if (res.success) {
          this.categories = res.data
        }
      },
    }
    ```

2. **store**  
    `Vuex.store` 实例，在运行时 `Nuxt.js` 会尝试找到根目录下 `store` 目录，如果该目录存在，它会将模块文件加到构建配置中。  
    该目录下 `index.js` 会被创建为根模块，其它文件为子模块。  
    ```js
    // /store/index.js
    export const state = () => ({
      list: [],
    })

    export const mutations = {
      updateList(state, paylaod) {
        state.list = paylaod
      },
    }
    ```
    `store` 会被同时注入客户端和服务端，可以这样使用：  
    ```js
    export default {
      async asyncData({ app, store }) {
        // 列表数据
        const list = await app.$axios.getIndexList({
          params: {
            pageNum: 1,
            pageSize: 20,
          },
        }).then(res => res.success ? res.data : [])
        // 服务端使用
        store.commit('updateList', list)
        return {
          list,
        }
      },
      data() {
        return {
          list: [],
        }
      },
      methods: {
        updateList(list) {
          // 客户端使用
          this.$store.commit('updateList', list)
        }
      },
    }
    ```
    关于 `store` 注入，首先在 `.nuxt/store.js` 中，对 `store` 模块文件作出一系列的处理，并暴露 `createStore` 方法。然后再 `.nuxt/index.js` 中，`createApp` 方法会对其双端同时注入：  
    ```js
    import { createStore } from './store.js'

    async function createApp (ssrContext) {
      const store = createStore(ssrContext)
      // ...
      // 注入到this
      const app = {
        store
      }
      // ...
      // 注入到context
      await setContext(app, {
        store
      })
      // ...
      return {
        store,
        app,
        router
      }
    }
    ```
    `Nuxt` 会通过 `inject` 方法为其挂载上 `plugin`（`plugin` 是挂载全局方法的主要途径），因此在 `store` 里，可以通过 `this` 访问到全局方法：
    ```js
    export const mutations = {
      updateList(state, paylaod) {
        console.log(this.$axios)
        state.list = paylaod
      }
    }
    ```

3. **params/query**  
    params 和 query 分别是 router.params 和 router.query 的别名，它们都带有路由参数的对象。
    ```js
    export default {
      async asyncData({ app, params }) {
        const list = await app.$axios.getIndexList({
          params: {
            id: params.id,
            pageNum: 1,
            pageSize: 20
          },
        }).then(res => res.success ? res.data : [])
        return {
          list
        }
      }
    }
    ```

4. **redirect**  
    重定向用户请求到另一个路由，通常会用在权限验证。`status`（状态码，默认为302），`status` 和 `query` 是可选参数。
    加入有个路由，需要验证登录身份，若身份过期重定向到登录页。  
    ```js
    if (!token) {
      // 单纯重定向路由，可以直接传入路径字符串
      redirect('/login')

      // 或者
      redirect(302, {
        path: '/login', // 路由路径
        query: { // 参数
          isExpires: 1,
        }
      })
      return
    }
    ```

5. **error**  
    该方法跳转到错误页。用法：`error(params)`，`params` 参数应该包含 `statusCode` 和 `message` 字段。  
    例如：商品详情页请求数据依赖 `query.id`，当 `query.id` 不存在时，跳转错误页。  
    ```js
    if (!query.id) {
      error({
        statusCode: 404,
        message: '商品不存在',
      })
      return
    }
    ```

#### 2. 页面常用生命周期
1. **asyncData**  
    > 在服务端异步获取并渲染数据  

    这是最常用最重要的生命周期，同时也是服务端渲染的关键点。该生命周期只限于页面组件调用，第一个参数为 `context` 。它调用的时机在组件初始化之前，运作在服务端环境。所以在 `asyncData` 生命周期中，无法通过 `this` 引用当前的 `Vue` 实例，也没有 `window` 对象和 `document` 对象。  
    一般在 `asyncData` 会对主要页面数据进行预先请求，获取到数据会交由服务端拼接成 `html` 返回前端渲染，以此提高首屏加载速度和进行 `seo` 优化。  
    ```js
    export default {
      async asyncData({ app }) {
        // 列表数据
        const list = await app.$axios.getIndexList({
          params: {
            pageNum: 1,
            pageSize: 20,
          },
        }).then(res => res.success ? res.data : [])
        return {
          list,
        }
      },
      data() {
        return {
          list: []
        }
      }
    }
    ```
    **注意：**  `asyncData` 只有首屏在服务端执行，其它时候相当于 `created` 或 `mounted` 在客户端渲染页面。  
    假如有两个页面，首页和详情页，都设置了 `asyncData`。进入首页，`asyncData` 运行在服务端。渲染完成后，点击跳转详情页，此时详情页的 `asyncData` 并不会在服务端执行，而是在客户端发起请求获取数据渲染，因为详情页已经不是首屏。当我们刷新详情页， `asyncData` 才会运行在服务端。

2. **fetch**  
    > fetch 方法用于在渲染页面前填充应用的状态（store）数据，与 asyncData 类似，不同的是它不会设置组件数据。

    为了让获取过程可以异步，需要返回一个 `Promise`，`Nuxt.js` 会等这个 `promise` 完成后再渲染组件。  
    ```js
    export default {
      fetch({ app, store }) {
        app.$axios.getData()
          .then(res => {
            store.commit('updateData', res.data)
          })
      }
    }
    ```
    或者使用 `async/await` 简化代码
    ```js
    export default {
      async fetch({ app, store }) {
        const { data } = await app.$axios.getData()
        store.commit('updateData', data)
      }
    }
    ```

3. **validata**  
    > 在动态路由对应的页面组件中配置一个校验方法用于校验动态路由参数的有效性。

    它的第一个参数是 `context`。与上面不同的是，我们可以访问实例上的方法 `this.methods.xxx`。  
    生命周期可以返回一个 `Boolean`，为真则进入路由，为假则停止渲染当前页面并显示错误页面：
    ```js
    export default {
      validate({ query }) {
        return this.methods.validateQuery(query.type)
      },
      methods: {
        validateQuery(type) {
          const typeWhiteList = ['backend', 'frontend', 'android']
          return typeWhiteList.includes(type)
        },
      }
    }
    ```
    或者返回一个Promise：
    ```js
    // TODO: 这里并不太清楚，实际效果是 404
    export default {
      validate() {
        return new Promise((resolve) => setTimeout(() => resolve()))
      }
    }
    ```
    还可以在验证函数执行期间抛出预期或意外错误：
    ```js
    export default {
      validate() {
        // 使用自定义消息触发内部服务器 500 错误
        throw new Error('Under Construction!')
      }
    }
    ```

4. **watchQuery**  
    > 监听参数字符串更改并在更改时执行组件方法（asyncData，fetch，validate，layout...）

    `watchQuery` 可设置 `Boolean` 或 `Array`（默认：[]）。使用 `watchQuery` 属性可以监听参数字符串的更改。如果定义的字符串发生变化，将调用所有组件方法（`asyncData`，`fetch`，`validate`，`layout`...）。为了提高性能，默认情况下禁用。
    ```js
    // 一个搜索结果页的例子
    export default {
      async asyncData({ app, query }) {
        let res = await app.$api
          .searchList({
            pageNum: 1,
            pageSize: 20,
            type: query.type ? query.type.toUpperCase() : "ALL",
            keyword: query.keyword
          })
          .then((res) => (res.success ? res.ddata : {}));
        return {
          pageInfo: res.pageInfo || {},
          searchList: res.edges || [],
        };
      },
      watchQuery: ['keyword', 'type'],
      methods: {
        search(item) {
          // 更新路由参数，触发 watchQuery，执行 asyncData 重新获取数据
          this.$router.push({
            name: "search",
            query: {
              type: item.type || this.type,
              keyword: this.keyword
            },
          });
        },
      },
    }
    ```
    使用 `watchQuery` 好处就是，当我们使用浏览器前进/后退按钮时，页面数据会刷新，因为参数字符串发生了变化。

5. **head**  
    > Nuxt.js 使用 vue-mate 更新应用头部标签（Head）和 html 属性。

    使用 `head` 方法设置当前页面的头部标签，该方法里能通过 `this` 获取组件数据。正确设置 `meta` 标签，有利于页面被搜索引擎查找，进行 `seo` 优化。
    ```js
    export default {
      head() {
        return {
          title: this.pruductInfo.title,
          meta: [
            { hid: 'description', name: 'description', content: this.pruductInfo.description },
            { hid: 'keywords', name: 'keywords', content: this.pruductInfo.keywords },
          ]
        }
      },
      data() {
        return {
          productInfo: {
            title: '《动物农场（反乌托邦小说经典）》([英]乔治·奥威尔)【摘要 书评 试读】- 京东图书',
            description: '京东JD.COM图书频道为您提供《动物农场（反乌托邦小说经典）》在线选购，本书作者：，出版社：中国友谊出版公司。买图书，到京东。网购图书，享受最低优惠折扣!',
            keywords: '动物农场（反乌托邦小说经典）,,中国友谊出版公司,9787505735156,,在线购买,折扣,打折',
          }
        }
      }
    }
    ```
    为了避免子组件中的 `meta` 标签不能正确覆盖父组件 中相同的标签而产生重复的现象，建议利用 `hid` 键为 `meta` 标签配一个唯一的标识编号。  
    在 `nuxt.config.js` 中，可以设置全局 `head`：
    ```js
    export default {
      head: {
        title: '京东(JD.COM)-正品低价、品质保障、配送及时、轻松购物！',
        meta: [
          { charset: 'utf-8' },
          { name: 'viewport', content: 'width=device-width, initial-scale=1' },
          { hid: 'description', name: 'description', content: '京东JD.COM-专业的综合网上购物商城,销售家电、数码通讯、电脑、家居百货、服装服饰、母婴、图书、食品等数万个品牌优质商品.便捷、诚信的服务，为您提供愉悦的网上购物体验!' },
          { hid: 'Keywords', name: 'Keywords', content: '网上购物,网上商城,手机,笔记本,电脑,MP3,CD,VCD,DV,相机,数码,配件,手表,存储卡,京东' }
        ]
      }
    }
    ```

6. **补充**  
    ```sh
    # 生命周期的调用顺序
    validata => asyncData => fetch => head
    ```

#### 3. 配置启动端口
默认在 3000 端口启动，可在 `nuxt.config.js` 中修改
```js
module.exports = {
  server: {
    port: 8080,
    host: '127.0.0.1'
  }
}
```

#### 4. 加载外部资源
1. **全局配置**  
    `nuxt.config.js`: 
    ```js
    module.exports = {
      head: {
        link: [
          { rel: 'stylesheet', href: '//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.18.1/build/styles/atom-one-light.min.css' },
        ],
        script: [
          { src: '//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.18.1/build/highlight.min.js' }
        ]
      }
    }
    ```

2. **组件配置**  
    组件内可在 `head` 配置  
    ```js
    export default {
      head() {
        return {
          link: [
            { rel: 'stylesheet', href: '//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.18.1/build/styles/atom-one-light.min.css' },
          ],
          script: [
            { src: '//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.18.1/build/highlight.min.js' }
          ]
        }
      }
    }
    ```

#### 5. 环境变量
1. **创建环境变量**  
    根目录创建 .env 文件管理环境变量无效，需要使用 `nuxt.config.js` 提供的 `env` 选项进行配置。
    ```js
    export default {
      env: {
        baseUrl: process.env.NODE_ENV === 'production' ? 'https://test.com' : 'http://localhost:3003'
      }
    }
    ```

2. **使用环境变量**  
    * `process.env.baseUrl`
    * `context.env.baseUrl`
    ```js
    // /plugins/axios.js
    export default function ({ app: { $axios } }) {
      $axios.defaults.baseURL = process.env.baseUrl
    }
    ```

#### 6. plugins
当你希望在整个应用程序中使用某个函数或者属性值，你需要将它们注入到 `Vue` 实例，`context`，甚至 `store（Vuex）`
```js
/**
 * context: 上下文对象
 * injexct: 该方法可以将 plugin 同时注入到 context，vue实例，vuex中
 */
export default function (context, inject) {}
```
1. **注入 Vue 实例**  
    定义：
    ```js
    // plugins/vue-main.js
    import Vue from 'vue'

    Vue.prototype.$myInjectedFunction = string => console.log('this is an example', string)
    ```
    使用：
    ```js
    // nuxt.config.js
    export default {
      plugins: [ '~/plugins/vue-main.js' ]
    }
    ```
    这样在所有vue组件中都可以使用该函数
    ```js
    export default {
      mounted() {
        this.$myInjectedFunction('test')
      }
    }
    ```
2. **注入 context**  
    定义：
    ```js
    // plugins/ctx-inject.js
    export default function ({ app }) {
      app.$myInjectedFunction = string => console.log('this is an example', string)
    }
    ```
    使用：
    ```js
    // nuxt.config.js
    export default {
      plugins: [ '~/plugins/ctx-inject.js' ]
    }
    ```
    现在，只要获取到 `context`，就可以使用该函数
    ```js
    export default {
      asyncData({ app }) {
        app.$myInjectedFunction('test')
      }
    }
    ```

3. **同时注入**  
    如需同时注入，可以使用 `inject` 方法，它是 `plugin` 导出函数的第二个参数。系统会默认将 `$` 作为方法名的前缀。  
    定义：  
    ```js
    // plugins/combined-inject.js
    export default ({ app }, inject) => {
      inject('myInjectedFunction', string => console.log('this is an example', string))
    }
    ```
    使用：  
    ```js
    // nuxt.config.js
    export default {
      plugins: ['~/plugins/combined-inject.js']
    }
    ```
    现在可以在 context，vue 实例中，或 vuex 中调用
    ```js
    export default {
      asyncData(context) {
        context.app.$myInjectedFunction('test')
      },
      mounted () {
        this.$myInjectedFunction('test')
      }
    }

    // store/index.js
    export const mutations = {
      updateList(state, paylaod) {
        this.$myInjectedFunction('test')
        state.list = paylaod
      },
    }
    ```

4. **plugin 相互调用**  
    当 plugin 依赖其他的 plugin 调用时，可以访问 context 来获取，前提是 plugin 需要使用 context 注入。  
    ```js
    // plugins/request.js
    export default ({ app: { $axios } }, inject) => {
      inject('request', {
        get(url, params) {
          return $axios({
            method: 'get',
            url,
            params
          })
        }
      })
    }
    ```
    ```js
    export default ({ app: { $request } }, inject) => {
      inject('api', {
        getIndexList(params) {
          return $request.get('/api/getList', params)
        }
      })
    }
    ```
    在注入 `plugin` 时要注意顺序，
    ```js
    module.exports = {
      plugins: [
        './plugins/axios.js',
        './plugins/request.js',
        './plugins/api.js',
      ]
    }
    ```

#### 7. 路由配置
在 `Nuxt` 中，路由是基于文件结构自动生成的，无需配置。自动生成的路由配置可在 `.nuxt/router.js` 中查看。  

1. **动态路由**  
    ```
    # 以下划线为前缀的 vue 文件或目录
    pages/
    --| users/
    -----| _id.vue
    --| index.vue
    ```
    对应生成的路由配置：
    ```js
    router:{
      routes: [
        {
          name: 'index',
          path: '/',
          component: 'pages/index.vue'
        },
        {
          name: 'users-id',
          path: '/users/:id?',
          component: 'pages/users/_id.vue'
        }
      ]
    }
    ```

2. **嵌套路由**  
    ```
    pages/
    --| users/
    -----| _id.vue
    -----| index.vue
    --| users.vue
    ```
    对应生成的路由配置：
    ```js
    router: {
      routes: [
        {
          path: '/users',
          component: 'pages/users.vue',
          children: [
            {
              path: '',
              component: 'pages/users/index.vue',
              name: 'users'
            },
            {
              path: ':id',
              component: 'pages/users/_id.vue',
              name: 'users-id'
            }
          ]
        }
      ]
    }
    ```
    在一级页面中使用 `nuxt-child` 来显示子页面，类似于 `router-view`
    ```vue
    <template>
      <div>
        <nuxt-child></nuxt-child>
      </div>
    </template>
    ```

3. **自定义配置**  
    通过修改 `nuxt.config.js` 中 `router` 选项来自定义，这些配置会被添加到 `Nuxt.js` 的路由配置中。  
    ```js
    module.exports = {
      router: {
        extendRoutes(routes, resolve) {
          routes.push({
            path: '/test',
            redirect: {
              name: 'error'
            }
          })
        }
      }
    }
    ```

#### 8. axios
在 `Nuxt` 项目初始化时可以直接选择集成 `@nuxt/axios`。也可以后期手动安装。
```sh
yarn add @nuxtjs/axios
```
```js
// nuxt.config.js
module.exports = {
  modules: [
    '@nuxtjs/axios'
  ],
}
```
1. **使用**  
    ```js
    export default {
      async asyncData({ app }) {
        const list = await app.$axios.getIndexList()
        return {
          list,
        }
      },
      data() {
        return {
          list: [],
          categories: [],
        }
      },
      async created() {
        const data = await this.$axios.getCategories()
        this.categories = data
      },
    }
    ```

2. **自定义配置 axios**  
    例如针对 `axios` 的 `baseUrl`、拦截器自定义配置，可以通过 `plugins` 引入。  
    ```js
    export default ({ app: { $axios } }) => {
      $axios.defaults.baseURL = process.env.baseUrl

      $axios.interceptors.request.use((config) => {
        return config
      })

      $axios.interceptors.response.use((response) => {
        const res = response.data
        switch (res.code) {
          case 200:
            return res
          default:
            console.warn(res.msg || '接口出错啦！')
            return Promise.reject(new Error(res.msg || 'Error'))
        }
      })
    }
    ```

#### 9. css 预处理器
以 `scss` 为例
1. **安装**  
    ```sh
    # 注意版本问题，高低版本可能出现不兼容问题
    yarn add node-sass@5.0.0 sass-loader@10.1.1 --dev
    ```
    无需配置，模板内直接使用
    ```vue
    <style lang="scss" scoped>
    .container {
      color: $theme;
    }
    </style>
    ```

2. **全局样式**  
    ```css
    /*  assets/css/global.scss */
    .main {
      width: 960px;
      margin: 0 auto;
      margin-top: 20px;
    }
    ```
    使用
    ```js
    module.exports = {
      css: [
        '~/assets/css/global.scss'
      ],
    }
    ```

3. **全局变量**  
    ```sh
    yarn add @nuxtjs/style-resources --dev
    ```
    定义
    ```scss
    // 字号
    $fs-h1: 22px;
    $fs-h2: 16px;
    ```
    使用
    ```js
    module.exports = {
      buildModules: [
        '@nuxtjs/style-resources'
      ],
      styleResources: {
        scss: [
          './assets/css/variables.scss'
        ]
      },
    }
    ```

### 二. 前端技术点
#### 1. asyncData 请求并行
`asyncData` 中多个异步请求没有相互依赖关系时，尽量并行请求，否则极易拖慢页面渲染，造成长时间白屏。  
使用 `Promise.all` 将请求并行发送。在项目封装基础请求时做好 `catch` 错误处理，确保所有请求都不会被 `reject`，或者在 `asyncData` 中对每一个请求都要捕获异常，避免因为某一个请求抛错，导致整个页面渲染出错。
```js
export default {
  async asyncData({ app }) {
    const [list, data] = await Promise.all([
      app.$api
        .getIndexList({
          first: 20,
          order: 'POPULAR',
          category: 1,
        })
        .then((res) => (res.success ? res.data : [])),
      app.$api.getOtherData().then((res) => (res.success ? res.data : '')),
    ])
    return {
      list,
      data,
    }
  },
}
```

#### 2. token 的设置与存储
在 `Nuxt` 中，部分请求在服务端发起，无法获取 `localStorage` 或 `sessionStorage`。  
借助 `cookie-universal-nuxt` 可以通过一致的 `api` 在前后端操作 `cookie`。  
1. **安装使用 cookie-universal-nuxt**  
    安装  
    ```sh
    yarn add cookie-universal-nuxt
    ```
    ```js
    module.exports = {
      modules: [
        'cookie-universal-nuxt'
      ],
    }
    ```
    使用
    ```js
    // 服务端
    app.$cookies.get('name') // 获取
    app.$cookies.set('name', 'value') // 设置
    app.$cookies.remove('name') // 删除

    // 客户端
    this.$cookies.get('name') // 获取
    this.$cookies.set('name', 'value') // 设置
    this.$cookies.remove('name') // 删除
    ```

2. **实际应用流程**  
    ```js
    // /utils/auth.js
    export const setAuthInfo = (ctx, res) => {
      let $cookies, $store
      if (process.client) {
        $cookies = ctx.$cookies
        $store = ctx.$store
      }
      if (process.server) {
        $cookies = ctx.app.$cookies
        $store = ctx.app.$store
      }
      if ($cookies && $store) {
        const expires = new Date(Date.now() + 8.64e7 * 365 * 10)
        // 设置 cookie
        $cookies.set('userId', res.userId, { expires })
        $cookies.set('userInfo', res.user, { expires })
        $cookies.set('token', res.token, { expires })
        // 设置 vuex
        $store.commit('auth/SET_USERID', res.userId)
        $store.commit('auth/SET_USERINFO', res.user)
        $store.commit('auth/SET_TOKEN', res.token)
      }
    }
    ```
    之后改造 `axios`，让它在请求时带上验证信息：
    ```js
    // /plugins/axios.js
    $axios.interceptors.request.use(
      (config) => {
        config.headers.token = $cookies.get('token') || ''
        return config
      },
      (error) => {
        return Promise.reject(error)
      }
    )
    ```

#### 3. 权限验证中间件
* 页面初次加载，中间件会在服务端执行。前端路由切换则在客户端执行（没有特别设置 `ssr: false` 的话），
* 会在页面渲染前执行，可以在此验证身份是否过期，和路由守卫相似
* 中间件中如果有异步代码，需要 `return` 一个 `promise`。
* 上下文 `context`，解构的属性是 `{ app, store }`，没有 `$`，好几次写错

1. **定义**  
    ```js
    // middleware/auth.js
    export default function (context) {
      const { app, store } = context
      const cookiesToken = app.$cookies.get('token')
      if (cookiesToken) {
        // 每次路由跳转，验证登录是否过期
        return app.$api.isAuth().then((res) => {
          if (res.s === 1) {
            if (res.d.isExpired) {
              app.$auth.removeAuthInfo(context)
            } else {
              const stateToken = store.state.suth.token
              if (cookiesToken && !stateToken) {
                // 一些操作可能会新开标签页导致 vuex 状态丢失，这里重新同步下 vuex 中的登录状态
                store.commit('auth/SET_USERID', app.$cookies.get('userId'))
                store.commit('auth/SET_USERINFO', app.$cookies.get('userInfo'))
                store.commit('auth/SET_TOKEN', app.$cookies.get('token'))
              }
            }
          }
        })
      }
    }
    ```

2. **使用**  
    ```js
    // nuxt.config.js 将中间件注入到每一个页面
    router: {
      middleware: ['auth'],
    },
    ```
    ```js
    // 在具体某个页面中配置中间件选项
    export default {
      middleware: ['auth'],
    }
    ```

#### 4. 组件注册管理
1. **自定义组件**  
    ```sh
    .
    ├── base    # 基础ui组件
    ├── common  # 通用业务组件
    └── pages   # 页面组件
    ```
    基础UI组件全局注册，并且加上私有前缀，其他组件在页面内手动引入
    ```js
    // nuxt.config.js
    // components: true 时，components路径下的组件会被自动注册，但出于优化考虑，进自动注册部分组件
    components: [
      {
        path: '~/components/base',
        prefix: 'd',
      }
    ],
    ```

2. **第三方组件库（ant-design-vue）**  
    ```sh
    yarn add babel-plugin-import --dev
    ```
    ```js
    // nuxt.config.js
    export default {
      build: {
        babel: {
          plugins: [
            [
              'import',
              {
                libraryName: 'ant-design-vue',
                libraryDirectory: 'es',
                style: 'css',
              },
            ],
          ],
        },
        transpile: [/ant-design-vue/], // 没有这个会报错
      },
    }
    ```
    ```js
    // components/antComponentInstall.js
    import { Button, DatePicker } from 'ant-design-vue'

    export default {
      install(Vue) {
        Vue.use(Button)
        Vue.use(DatePicker)
      },
    }
    ```
    ```js
    // plugins/vue-main.js
    import antComponentInstall from '~/components/antComponentInstall'
    export default () => {
      Vue.use(antComponentInstall)
    }
    ```
    尽管组件按需引入，通过代码体积分析，发现icon仍然会被一次性全部引入。网上找到一个可用的方案，但比较麻烦，目前项目中并没有使用，等待官方的解决方案。
    ```js
    // plugins/ant-icons.js
    // 只将项目中使用的icon手动引入，问题在于ant组件使用的icon也来自这里，忘记引入的话，会导致组件异常
    export {
      InfoCircleFill,
      DownOutline,
      UpOutline,
    } from '@ant-design/icons'
    ```
    ```js
    // nuxt.config.js
    build: {
      extend(config) {
        config.resolve.alias['@ant-design/icons/lib/dist$'] = path.resolve(__dirname, './plugins/ant-icons.js')
      },
    },
    ```

#### 5. 页面布局切换
在构建网站应用时，大多数页面的布局都会保持一致。但在某些需求中，可能需要更换另一种布局方式，可借助页面 `layout` 配置项完成。  

1. **定义**  
    ```html
    <!-- layouts/default.vue -->
    <template>
      <div>
        <SiteNav />
        <Nuxt />
        <SiteFooter />
      </div>
    </template>

    <script>
    import SiteNav from '~/components/layouts/site-nav.vue'
    import SiteFooter from '~/components/layouts/site-footer.vue'

    export default {
      components: { SiteNav, SiteFooter },
    }
    </script>
    ```
    ```html
    <!-- layouts/empty.vue -->
    <template>
      <Nuxt />
    </template>
    ```

2. **使用**  
    ```js
    export default {
      layout: 'empty',
      // 或
      layout(context) {
        return 'empty'
      }
    }
    ```

#### 6. 自定义错误页
* 需放在 `layouts` 目录中，且名为 `error`。文件内不需包含 `<nuxt/>` 标签。
* 错误页中 `props` 接受一个 `error` 对象，该对象至少包含两个属性 `statusCode` 和 `message`。除此之外，还可以传其他属性。
    ```js
    export default {
      async asyncData({ query, error }) {
        if (!query.id) {
          error({
            statusCode: 404,
            message: '标签不存在',
            query,
          })
          return
        }
        return {
          tagInfo: {}
        }
      }
    }
    ```
* 若抛出错误，`message` 是 `new Error` 中的内容。如果想传对象过去，`message` 会转为字符串 `[object Object]`，可以使用 `JSON.stringify`。
    ```js
    export default {
      async validate ({ params }) {
        throw new Error(JSON.stringify({
          message: 'validate错误',
          params,
        }))
      }
    }
    ```

#### 7. 封装触底事件
```js
// mixins/reachBottom.js
export default {
  data() {
    return {
      _isReachBottom: false, // 防止进入执行区域时重复触发
      reachBottomDistance: 100, // 距离底部多远触发
    }
  },
  mounted() {
    this._scrollElement = document.scrollingElement
    this._throttled = this.$utils.throttle(this._windowScrollHandler, 200)
    window.addEventListener('scroll', this._throttled)
  },
  beforeDestroy() {
    window.removeEventListener('scroll', this._throttled)
  },
  methods: {
    _windowScrollHandler() {
      const scrollHeight = this._scrollElement.scrollHeight
      const currentHeight =
        this._scrollElement.scrollTop +
        this._scrollElement.clientHeight +
        this.reachBottomDistance
      if (currentHeight < scrollHeight && this._isReachBottom) {
        this._isReachBottom = false
      }
      if (this._isReachBottom) return
      if (currentHeight >= scrollHeight) {
        this._isReachBottom = true
        typeof this.reachBottom === 'function' && this.reachBottom()
      }
    },
  },
}
```

#### 8. 命令式弹窗组件

### 三. 中间层技术点
代理、缓存、日志、监控、数据处理。中间层的存在让前后端职责分离更加彻底。
#### 1. 请求转发
1. **相关中间件**  
    ```sh
    yarn add koa-router koa-bodyparser
    ```
    * `koa-router`: 路由中间件，快速定义路由及管理路由
    * `koa-bodyparser`: 参数解析中间件，支持解析json、表单类型

2. **路由设计**  
    [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
    * 路由目录：`/app/routers/v1`
    * 路由路径：标签相关接口文件 `tags`；用户相关接口文件 `users`
        > 在RESAful 架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表名对应。一般来说，数据库表都是同种记录的“集合”，所以 API 中的名词也应该使用复数。
    * 路由类型：由 `HTTP` 动词表示
        * GET(SELECT)：从服务器取出资源（一项或多项）
        * POST(CREATE)：在服务器新建一个资源
        * PUT(UPDATE)：在服务器更新资源（客户端提供改变后的完整资源）
        * DELETE(DELETE)：从服务器删除资源

3. **路由逻辑**  
    ```js
    // app/routers/users.js
    const Router = require('koa-router')
    const router = new Router()

    /**
    * @param {String} userId - 用户id
    */
    router.get('/userInfo', (ctx) => {
      ctx.body = {
        data: {
          name: 'mao',
          age: 18,
        },
      }
    })

    module.exports = router
    ```
    
4. **注册路由**  
    ```js
    // /app/index.js
    function useRouter(){
      let module = require('./routes/users')
      router.use('/v1/users', module.routes())
      app.use(router.routes()).use(router.allowedMethods())
    }
    ```

#### 2. 路由自动化注册
以 `routers` 作为路由主目录，向下寻找 `js` 文件注册路由，最终以 `js` 文件路径作为路由名。  
例如：`/app/routers/v1/users.js` 中有用户信息接口 `/userInfo`，最终接口调用地址为 `localhost:3000/v1/users/userInfo`  
```js
const fs = require('fs')
const path = require('path')

function useRouter(realPath = __dirname) {
  const urls = fs.readdirSync(realPath)
  urls.forEach((element) => {
    const elementPath = path.join(realPath, element)
    const stat = fs.lstatSync(elementPath)
    const isDir = stat.isDirectory()
    if (isDir) {
      useRouter(elementPath)
    } else {
      const module = require(elementPath)
      // 正确的路由文件应该返回 koa-router 实例
      if (module.routes) {
        const routerPrefix = realPath.split('/routers')[1] || ''
        // routers 里的文件路径作为 路由名
        router.use(
          path.join(routerPrefix, element.replace('.js', '')),
          module.routes()
        )
      }
    }
  })
}

// 使用路由
useRouter()
app.use(router.routes()).use(router.allowedMethods())
```

#### 3. 统一返回格式 & 错误处理
所有的返回值在 `app/middlewares/error.js` 里拦截一下，如果状态码是 200，用成功的工具函数包装返回，否则两种情况：一是自己业务抛错，使用失败工具函数包装返回；另一种是程序运行时报错，触发koa错误处理事件去处理
```js
// app/extend/context.js
const context = async (ctx, next) => {
  ctx.res.fail = ({ code, data, message }) => {
    ctx.body = {
      code,
      data,
      message,
    }
  }

  ctx.res.success = (message) => {
    ctx.body = {
      code: 0,
      data: ctx.body,
      message: message || 'success',
    }
  }

  await next()
}

module.exports = context
```
```js
// app/middlewares/response.js
const response = () => {
  return async function (ctx, next) {
    try {
      await next()
      if (ctx.status === 200) {
        ctx.res.success()
      }
    } catch (err) {
      if (err.code) {
        ctx.res.fail({ code: err.code, message: err.message })
      } else {
        ctx.app.emit('error', err, ctx)
      }
    }
  }
}

module.exports = response
```
在`app/middlewares/index.js` 文件中引入上面两个中间件：
```js
const bodyParser = require('koa-bodyparser')
const response = require('./response')
const error = require('./error')

/**
 * 参数解析
 */
const mdBodyParser = bodyParser({
  enableTypes: ['json', 'form', 'text', 'xml'],
})

/**
 * 统一返回格式
 */
const mdResHandler = response()

/**
 * 错误处理
 */
const mdErrorHandler = error()

module.exports = [mdResHandler, mdErrorHandler, mdBodyParser]
```
启动文件`app/index.js`中，添加全局错误捕获
```js
app.on('error', (err, ctx) => {
  if (ctx) {
    ctx.body = {
      code: 9999,
      message: `程序运行时报错：${err.message}`,
    }
  }
})
```
添加完处理程序后，验证下返回结果：
```js
router.get('/userInfo', (ctx) => {
  ctx.body = {
    text: '返回结果',
  }
})

// {
//   "code": 0,
//   "data": {
//     "text": "返回结果"
//   },
//   "message": "success"
// }
```
```js
router.get('/userInfo', (ctx) => {
  ctx.utils.assert('', ctx.utils.throwError(1001, '验证码失败'))
})

// {
//   "code": 1001,
//   "message": "验证码失败"
// }
```
```js
router.get('/userInfo', (ctx) => {
  const b = a
  ctx.body = {
    text: '返回结果',
  }
})

// {
//   "code": 9999,
//   "message": "程序运行时报错：a is not defined"
// }
```

#### 4. 路由参数验证
基于 `async-validator` 封装一个工具函数，挂载到 `ctx` ，提前对参数验证，终止错误查询并告知使用者。
```js
const { default: Schema } = require('async-validator')

module.exports = async (ctx, next) => {
  ctx.validator = async (descriptor) => {
    const validator = new Schema(descriptor)
    const params = {}
    // 获取参数
    Object.keys(descriptor).forEach((key) => {
      if (ctx.method === 'GET') {
        params[key] = ctx.query[key]
      } else if (
        ctx.method === 'POST' ||
        ctx.method === 'PUT' ||
        ctx.method === 'DELETE'
      ) {
        params[key] = ctx.request.body[key]
      }
    })

    // 验证参数
    const errors = await validator
      .validate(params)
      .then(() => null)
      .catch((err) => err.errors)
    if (errors) {
      // 验证不通过
      const error = new Error(JSON.stringify(errors))
      error.code = 9998
      throw error
    }
  }

  await next()
}
```
使用：
```js
// app/controller/users.js
module.exports = {
  userList: async (ctx) => {
    await ctx.validator({
      udid: { type: 'string', required: true },
      pageSize: {
        type: 'string',
        required: true,
        validator: (rule, value) => Number(value) > 0,
        message: 'limit 需传入正整数',
      },
      order: { type: 'enum', enum: ['rankIndex', 'createdAt'] },
    })

    ctx.body = {
      list: [],
    }
  },
}

```
`type` 代表参数类型，`require` 代表是否必填。当 `type` 为 `enum` （枚举）类型时，参数值只能为 `enum` 数组中的某一项。  
```js
{
  "code": 9998,
  "message": "[{\"message\":\"udid is required\",\"field\":\"udid\"},{\"message\":\"pageSize 需传入正整数\",\"field\":\"pageSize\"}]"
}
```

#### 5. 跨域设置
```sh
yarn add koa2-cors
```
如果不符合请求的方式，或带有未允许的标头。发送请求时会直接失败，浏览器抛出 `cors` 策略限制的错误。
```js
const cors = require('koa2-cors')

const mdCors = cors({
  origin(ctx) {
    return ctx.request.header.origin
  },
  credentials: true,
  allowMethods: ['OPTIONS', 'GET', 'PUT', 'POST', 'DELETE'],
  allowHeaders: ['Content-Type', 'X-Requested-With', 'token'],
  maxAge: 86400,
})
module.exports = [mdCors, mdResHandler, mdErrorHandler, mdBodyParser]
```

#### 6. 网站安全性
`koa-helmet` 提供重要的安全标头，使你的应用 程序在默认情况下更加安全。
```sh
yarn add koa-helmet
```
```js
const helmet = require('koa-helmet')
const mdHelmet = helmet()

module.exports = [mdCors, mdHelmet, mdResHandler, mdErrorHandler, mdBodyParser]
```
默认为我们做了以下安全设置：
* `Content-Security-Policy`: 内容安全策略（CSP），用于检测并削弱某些特定类型的攻击。
* `X-DNS-Prefetch-Control`: 禁用浏览器的 DNS 预取
* `X-Frame-Options`: 确保自己的网站 没有被嵌到别人的网站中去，避免点击劫持攻击
* `X-Powered-By`: 表用用于支持当前网页应用程序的技术
* `Strict-Transport-Security`:  它告诉浏览器只能使用 `HTTPS` 访问当前资源
* `X-DownLoad-Options`: 防止 IE 在在站点上下文中执行下载
* `X-Content-Type-Options`: 设置为 `nosniff`，有助于防止浏览器试图猜测 MINE 类型，这可能会带来安全隐患。
* `X-XSS-Protection`: 防止反射的 XSS 攻击
* ... 

#### 7. 缓存
基于 LRU-Cache
1. **页面缓存**  

2. **组件缓存**  

3. **API缓存**  

#### 8. 限流
1. **单 IP 限流**  

2. **IP 黑名单**  

#### 9. 灾备
高并发或异常情况下，SSR降级为CSR
1. **监控系统降级**  
    监测 Node 进程的CPU和内存使用率，设定阈值

2. **全平台降级**  
    大促期间，提前降级

3. **单次访问降级**  
    偶发性异常，自动切换CSR

4. **指定渲染方式**  
    url中增加参数isCsr=true，Nginx 层对参数isCsr进行拦截分流

#### 10. 日志
log4js
1. **页面渲染日志**  

2. **接口日志**  

3. **中间层接口日志**  

4. **日志采集**  

#### 11. 流量监控

#### 12. 错误异常监控

### 四. 其它
#### 1. 在 VS Code 中调试 server 端代码
1. VS Code 安装 `Debugger for Chrome` 插件
2. `package.json` 中点击 npm script 命令上方的`调试`
3. 选择 `dev` 命令，启动服务与调试
4. 代码中加入 `debugger`，刷新浏览器
5. 之后就可以快乐的断点调试了

#### 2. 配置ip访问
```json
// package.json
"scripts": {
  "dev": "nuxt --hostname 0.0.0.0 --port 3000"
}
```

#### 3. 开发中关闭 eslint
Nuxt 初始化如果选择添加 lint 之后，默认开启保存代码时校验，别提多烦人了，严重影响开发效率。而且没有像 vue-cli 一样提供 `lintOnSave` 关闭选项。  
为了关掉这个破东西，我花了一年时间...，最后我放弃了
```js
// nuxt.config.js
export default {
  buildModules: [
    '@nuxtjs/eslint-module',
    '@nuxtjs/stylelint-module',
  ]
}
```
删掉这两项，只保留 `githook` 校验，世界都清净了。

#### 4. 使用 vconsole
1. 安装  
    ```sh
    yarn add vconsole --dev
    ```
2. 创建插件  
    ```js
    // plugins/vconsole.js
    // TODO: 使用import引入会被打包到生产环境中，需要研究下
    const VConsole =
      process.env.NODE_ENV !== 'production' ? require('vconsole') : ''
    export default () => {
      if (process.env.NODE_ENV !== 'production') {
        window.vConsole = new VConsole()
      }
    }
    ```
3. 使用插件  
    ```js
    export default {
      plugins: [
        {
          src: '~/plugins/vconsole',
          ssr: false,
        }
      ]
    }
    ```
4. 报错
    使用插件后，直接打印vue的根级属性，会报错，原因没有深究
    ```js
    console.log(this.$store)

    [Vue warn]: Property or method "toJSON" is not defined on the instance but referenced during render.
    ```

### 五. 引用
[🚀点亮你的Vue技术栈，万字Nuxt.js实践笔记来了](https://juejin.cn/post/6844904160324747278#heading-50)

[抄得走, 用得到的 Koa 实践总结](https://mp.weixin.qq.com/s/sjtwDEOwU8uLtxiAqwi7ug)

[开发效率提升50%以上，爱奇艺官网主站的Nuxt实践](https://mp.weixin.qq.com/s/H-2Dfw_UCQBIKnRbdPLNlQ)

TODO: 待整理
[Nuxt.js 利用 Nginx 实现同构降级策略](https://linhong.me/2021/03/04/nuxt-ssr-to-csr/)

[SSR同构降级策略](https://juejin.cn/post/6884107649843986440#heading-5)

[vivo 商城架构升级-SSR 实战篇](https://segmentfault.com/a/1190000038575056)
