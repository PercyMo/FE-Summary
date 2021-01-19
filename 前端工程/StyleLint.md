## StyleLint配置

### 一. 实践
#### 1. 安装
```sh
npm install stylelint stylelint-config-recess-order stylelint-order stylelint-config-standard --save-dev
```
* [stylelint](https://stylelint.io/)：规则检查工具
* [stylelint-config-standard](https://github.com/stylelint/stylelint-config-standard#readme)：关于规范、风格约束的规则集，其中已经做了许多规则的配置
* stylelint-order：对 css 属性进行排序的插件工具
* stylelint-config-recess-order：社区提供的属性排序的规则集

#### 2. 配置
```js
// .stylelintrc.js
module.exports = {
  'extends': ['stylelint-config-standard', 'stylelint-config-recess-order'],
  'plugins': ['stylelint-scss'],
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
    ],
    "*.{html,vue,css,sass,scss}": [
      "stylelint --fix"
    ]
  }
}
```

### 二. 总结
css属性推荐的书写顺序
1. 定位相关属性 `position`、`z-index`、`top`
2. 自身相关属性 `width`、`padding`、`margin`、`boder`
3. 文字样式 `font`、`color`
4. 文本属性 `text-align`、`word-spacing`
5. css3新增属性

### 三. 引用
[如何为你的 Vue 项目添加配置 Stylelint](https://juejin.im/post/5c31c9a16fb9a049f8197000)