## StyleLint配置

### 一. 实践
#### 1. 安装
```sh
yarn add -D stylelint stylelint-scss postcss postcss-html postcss-scss stylelint-config-html stylelint-config-prettier stylelint-config-recommended stylelint-config-recommended-vue stylelint-config-standard stylelint-order
```
* [stylelint](https://stylelint.io/)：规则检查工具
* [stylelint-config-standard](https://github.com/stylelint/stylelint-config-standard#readme)：关于规范、风格约束的规则集，其中已经做了许多规则的配置
* stylelint-order：对 css 属性进行排序的插件工具

#### 2. 配置
```js
// stylelint.config.js
module.exports = {
  extends: [
    'stylelint-config-standard',
    'stylelint-config-prettier',
    'stylelint-config-html/vue',
    'stylelint-config-recommended-vue',
  ],
  plugins: ['stylelint-order'],
  customSyntax: 'postcss-html',
  ignoreFiles: ['**/*.js', '**/*.jsx', '**/*.tsx', '**/*.ts', '**/*.json'],
  rules: {
    'selector-class-pattern': null,
    indentation: 2,
    'selector-pseudo-element-no-unknown': [
      true,
      {
        ignorePseudoElements: ['v-deep', ':deep'],
      },
    ],
    // scss 在校验时对部分关键字不识别会报错，没看到有好的解决方案，暂时屏蔽掉该条规则
    'at-rule-no-unknown': [
      true,
      {
        ignoreAtRules: ['function', 'if', 'each', 'include', 'mixin'],
      },
    ],
    'order/properties-order': [
      'position',
      'top',
      'right',
      'bottom',
      'left',
      'z-index',
      'display',
      'justify-content',
      'align-items',
      'float',
      'clear',
      'overflow',
      'overflow-x',
      'overflow-y',
      'margin',
      'margin-top',
      'margin-right',
      'margin-bottom',
      'margin-left',
      'padding',
      'padding-top',
      'padding-right',
      'padding-bottom',
      'padding-left',
      'width',
      'min-width',
      'max-width',
      'height',
      'min-height',
      'max-height',
      'font-size',
      'font-family',
      'font-weight',
      'border',
      'border-style',
      'border-width',
      'border-color',
      'border-top',
      'border-top-style',
      'border-top-width',
      'border-top-color',
      'border-right',
      'border-right-style',
      'border-right-width',
      'border-right-color',
      'border-bottom',
      'border-bottom-style',
      'border-bottom-width',
      'border-bottom-color',
      'border-left',
      'border-left-style',
      'border-left-width',
      'border-left-color',
      'border-radius',
      'text-align',
      'text-justify',
      'text-indent',
      'text-overflow',
      'text-decoration',
      'white-space',
      'color',
      'background',
      'background-position',
      'background-repeat',
      'background-size',
      'background-color',
      'background-clip',
      'opacity',
      'filter',
      'list-style',
      'outline',
      'visibility',
      'box-shadow',
      'text-shadow',
      'resize',
      'transition',
    ],
  },
};
```

#### 3. 结合 Git Hooks 使用
```js
// lint-staged.config.js
// 与 eslint、prettier 结合使用，可以只关注 stylelint 部分
module.exports = {
  '*.{js,jsx,ts,tsx}': ['eslint --fix --max-warnings=0', 'prettier --write'],
  '*.json': ['prettier --write'],
  '*.vue': [
    'eslint --fix --max-warnings=0',
    'prettier --write',
    'stylelint --fix',
  ],
  '*.{scss,less,styl,html}': ['stylelint --fix', 'prettier --write'],
  '*.md': ['prettier --write'],
};

```

### 二. 总结
css属性推荐的书写顺序
1. 定位相关属性 `position`、`z-index`、`top`
2. 自身相关属性 `width`、`padding`、`margin`、`boder`
3. 文字样式 `font`、`color`
4. 文本属性 `text-align`、`word-spacing`
5. css3新增属性
