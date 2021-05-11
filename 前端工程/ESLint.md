## ESLint详解

### 一. ESLint原理

### 二. ESLint配置
#### 1. 安装
自己的项目是基于 vue 的，部分依赖脚手架初始化的模板中已经安装过了，总之确认以下依赖包被正确安装了。
```json
// 版本号是从当时项目中粘出来的，及时更新
{
  "devDependencies": {
    "@vue/cli-plugin-eslint": "~4.5.0",
    "babel-eslint": "^10.1.0",
    "eslint": "^7.17.0",
    "eslint-plugin-vue": "^7.4.1",
    "standard": "^16.0.3"
  }
}
```

#### 2. 配置
`vue-cli` 初始化的模板中，把部分 `eslint` 配置放到了 `package.json` 中，建议全部移到 `.eslintrc.js` 中管理
```js
module.exports = {
  root: true,
  env: {
    browser: true,
    node: true,
    es2021: true
  },
  extends: [
    'eslint:recommended',
    'plugin:vue/recommended',
    'standard'
  ],
  parserOptions: {
    ecmaVersion: 12,
    sourceType: 'module',
    parser: 'babel-eslint'
  },
  plugins: [
    'vue'
  ],
  rules: {
    // 多个特性(3个以上)的元素应该分多行撰写，每个特性一行
    'vue/max-attributes-per-line': [1, {
      singleline: 3,
      multiline: {
        max: 1,
        allowFirstLine: false
      }
    }],
    // vue 模板使用2空格缩进
    'vue/html-indent': ['error', 2],
    indent: ['error', 2]
  }
}
```

#### 3. 结合 Git Hooks 使用
```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{js,vue}": [
      "eslint --fix"
    ]
  }
}
```
项目中发现，mac 使用 sourceTree 提交代码会自动跳过hook，可以做如下处理：
```sh
vim .git/hooks/pre-commit
PATH=$PATH:/usr/local/bin:/usr/local/sbin
```


#### 4. lint 时会漏掉部分特殊文件夹
项目中发现 `.xx` 这种文件，在lint校验时会默认被忽略掉。
```
# .eslintignore 

!docs/.vuepress/**
```
执行lint脚本时，加上 `--no-ignore` 参数
```sh
eslint --fix --no-ignore
```

### 四. 引用
