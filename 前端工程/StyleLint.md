## StyleLint配置

### 一. 实践
#### 1. 安装
```sh
"postcss": "^8.4.16",
"postcss-html": "^1.5.0",
"postcss-scss": "^4.0.4",
"stylelint": "^15.4.0",
"stylelint-config-property-sort-order-smacss": "^9.1.0",
"stylelint-config-recommended": "^11.0.0",
"stylelint-config-recommended-scss": "^9.0.1",
"stylelint-config-recommended-vue": "^1.4.0",
"stylelint-config-standard": "^32.0.0",
"stylelint-config-standard-scss": "^7.0.1",
"stylelint-order": "^6.0.3",
"stylelint-prettier": "^3.0.0",
```
* [stylelint](https://stylelint.io/)：规则检查工具
* [stylelint-config-standard](https://github.com/stylelint/stylelint-config-standard#readme)：关于规范、风格约束的规则集，其中已经做了许多规则的配置
* stylelint-order：对 css 属性进行排序的插件工具

#### 2. 配置
```js
// stylelint.config.js
module.exports = {
  extends: ['stylelint-config-standard', 'stylelint-config-property-sort-order-smacss'],
  plugins: ['stylelint-order', 'stylelint-prettier'],
  // customSyntax: 'postcss-html',
  overrides: [
    {
      files: ['**/*.(css|html|vue)'],
      customSyntax: 'postcss-html',
    },
    {
      files: ['*.less', '**/*.less'],
      customSyntax: 'postcss-less',
      extends: ['stylelint-config-standard', 'stylelint-config-recommended-vue'],
    },
    {
      files: ['*.scss', '**/*.scss'],
      customSyntax: 'postcss-scss',
      extends: ['stylelint-config-standard-scss', 'stylelint-config-recommended-vue/scss'],
      rule: {
        'scss/percent-placeholder-pattern': null,
      },
    },
  ],
  rules: {
    'selector-not-notation': null,
    'import-notation': null,
    'function-no-unknown': null,
    'selector-class-pattern': null,
    'selector-pseudo-class-no-unknown': [
      true,
      {
        ignorePseudoClasses: ['global', 'deep'],
      },
    ],
    'selector-pseudo-element-no-unknown': [
      true,
      {
        ignorePseudoElements: ['v-deep', ':deep'],
      },
    ],
    'at-rule-no-unknown': [
      true,
      {
        ignoreAtRules: [
          'tailwind',
          'apply',
          'variants',
          'responsive',
          'screen',
          'function',
          'if',
          'each',
          'include',
          'mixin',
          'extend',
        ],
      },
    ],
    'no-empty-source': null,
    'string-quotes': null,
    'named-grid-areas-no-invalid': null,
    'no-descending-specificity': null,
    'font-family-no-missing-generic-family-keyword': null,
    'rule-empty-line-before': [
      'always',
      {
        ignore: ['after-comment', 'first-nested'],
      },
    ],
    'unit-no-unknown': [true, { ignoreUnits: ['rpx'] }],
    'order/order': [
      [
        'dollar-variables',
        'custom-properties',
        'at-rules',
        'declarations',
        {
          type: 'at-rule',
          name: 'supports',
        },
        {
          type: 'at-rule',
          name: 'media',
        },
        'rules',
      ],
      { severity: 'error' },
    ],
  },
  ignoreFiles: ['**/*.js', '**/*.jsx', '**/*.tsx', '**/*.ts'],
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
