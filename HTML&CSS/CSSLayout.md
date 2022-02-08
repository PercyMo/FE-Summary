## CSS经典布局

### 一. 垂直水平居中
#### 1. Flexbox 中实现水平垂直居中
1. **Flex容器和Flex项目上设置对齐方式**
    1. 这种方式特别适合让 Icon 图标在容器中水平垂直居中，如果容器仍希望一行显示，可以设置 `display: inline-flex`
        ```html
        <div class="flex__container">
            <div class="flex__item"></div>
        </div>
        ```
        ```css
        .flex__container {
            display: flex; /* 或 inline-flex */
            justify-content: center;
            align-items: center;
        }
        ```
    2. 多个元素垂直水平居中，可以设置 `flex-direction: column;`
        ```html
        <div class="flex__container">
            <div class="flex__item"></div>
            <div class="flex__item"></div>
            <div class="flex__item"></div>
        </div>
        ```
        ```css
        .flex__container {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }

        /* 或者 */
        .flex__container {
            display: flex;
            flex-direction: column;
            justify-content: center;
        }
        .flex__item {
            align-self: center;
        }

        /* 再或者使用 palce-content: center; 实现 */
        .flex__container {
            display: flex;
            place-content: center; /* 相当于 align-content 和 justify-content 的简写 */
            place-items: center; /* 相当于 align-items 和 justify-items 的简写 */
            /* flex-wrap: wrap; 所有 flex 子项只有一行，则 align-content 不生效，所以也可以换用 wrap 来居中 */
        }
        
        /* 再再或者 */
        .flex__container {
            display: flex;
            place-content: center;
        }
        .flex__item {
            align-self: center;
        }
        ```

2. **在 Flex 项目上设置 `margin: auto;`**
    Flex 容器中只有一个项目，通过设置 `margin: auto;`，垂直水平居中
    ```css
    .flex__container {
        display: flex; /* 或者 inline-flex; */
    }
    .flex__item {
        margin: auto auto auto auto; /* 你还可以通过调整4个 auto 的值来让 item 处于不同的位置 */
    }
    ```

#### 2. Grid 中实现水平垂直居中
没有显式地在容器上设置`grid-template`，浏览器会将 Grid 容器默认设置为 Grid 内容大小
```css
.grid__container {
  display: grid;
  place-items: center;
}
```

### 二. 等高布局
```html
<div class="container">
  <div class="item">item</div>
  <div class="item">item</div>
  <div class="item">item</div>
</div>
```
```css
/* Flexbox 中实现 */
.container {
  display: flex; /* 或者 inline-flex */
}

/* Grid 中实现 */
.container {
  display: grid;
  grid-template-columns: 20vw 1fr 20vw; /* 根据需求调整值 */
}
```

### 三. Sticky Footer
```html
<header></header>
<main></main>
<footer></footer>
```
Sticky Footer 效果是：当内容不足一屏时，`<footer>`会在页面最底部；当内容超出一屏时，`<footer>`会自动往后延后。

```css
/* TODO: 个人试验，还需要撑开html和body高度才可以实现效果 */
html {
  min-height: 100%;
  display: flex;
}
body {
  flex: 1;
}

/* Flexbox 中实现 */
body {
  display: flex;
  flex-direction: column;
}

/* main 扩展撑开内容 */
main {
  flex-grow: 1;
}
/* 或者 footer 向上撑开内容都可以 */
footer {
  margin-top: auto;
}


/* Grid 中实现 */
/* 此处和 flex 一样，也需要先撑开html高度才可以实现效果 */
body {
  display: grid;
  grid-template-rows: auto 1fr auto;
}
```

### 四. 均分列
```html
<div class="container">
  <div class="column"></div>
  <div class="column"></div>
  <div class="column"></div>
</div>
```
```css
/* Flexbox 中实现 */
.container {
  display: flex;
}
.column {
  flex: 1;
}

/* Grid 中实现 */
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}
```

### 五. 圣杯布局
上古时代通过各种稀奇古怪方式实现的三栏布局就不看了，还有双飞翼，拥抱 `flex` 和 `grid` 吧！
#### 1. flex 实现
```html
<header>header</header>
<main>
  <article>article</article> <!-- 主内容 -->
  <nav>nav</nav>
  <aside>aside</aside>
</main>
<footer>footer</footer>
```
```css
body {
  width: 100vw;
  display: flex;
  flex-direction: column;
}

main {
  flex: 1;
  min-height: 0;
  display: flex;
  align-items: stretch;
  width: 100%;
}
article {
  flex: 1;
}
nav {
  width: 220px;
  order: -1;
}
aside {
  width: 220px;
}
footer {
  margin-top: auto;
}

/* 响应式 */
@media screen and (max-width: 800px) {
  main {
    flex-direction: column;
  }
  nav, aside {
    width: 100%;
  }
}
```

#### 2. grid 实现
```html
<header>header</header>
<main>main</main>
<nav>nav</nav>
<aside>aside</aside>
<footer>footer</footer>
```
```css
body {
  display: grid;
  grid-template-columns: 220px 1fr 220px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas: 
    "header header header"
    "nav main aside"
    "footer footer footer";
}

header {
  grid-area: header;
}
main {
  grid-area: main;
}
nav {
  grid-area: nav;
}
aside {
  grid-area: aside;
}
footer {
  grid-area: footer;
}

@media screen and (max-width: 800px) {
  body {
    grid-template-columns: 1fr;
    grid-template-rows: auto auto 1fr auto auto;
    grid-template-areas: 
      "header"
      "nav"
      "main"
      "aside"
      "footer";
  }
}
```

### 六. 12列网格布局
```html
<div class="row">
  <div class="item col1">
    <div>col1</div>
  </div>
  <div class="item col2">
    <div>col2</div>
  </div>
  <div class="item col3">
    <div>col3</div>
  </div>
  <div class="item col4">
    <div>col4</div>
  </div>
  <div class="item col2">
    <div>col2</div>
  </div>
</div>
```
#### 1. flex 实现
参考文章中的布局写法，利用 `flex-grow:1` 属性撑开剩余空间，以及将 `gutter` 计算在 `flex-basis` 中的写法，个人实际测试发现宽度并不精确，且不太优雅，计算逻辑比较绕。因此以下为参考 antd 和 element 的栅格布局写法。

```css
:root {
  --gutter: 20px;
  --column: 12;
  --span: 1;
}

.row {
  display: flex;
}
.item {
  flex: 0 0 calc(100% / var(--column) * var(--span));
  padding: 0 calc(var(--gutter) / 2);
  box-sizing: border-box; /* 将 padding 排除在宽度计算之外 */
}

.col1 {
  --span: 1;
}
.col2 {
  --span: 2;
}
.col3 {
  --span: 3;
}
.col4 {
  --span: 4;
}
/* ... */
```

#### 2. grid 实现
```css
:root {
  --gutter: 20px;
  --column: 12;
  --span: 1;
}

.row {
  display: grid;
  grid-template-columns: repeat(var(--column), 1fr); /* 均分为12列 */
  grid-template-rows: 1fr;
}
.item {
  padding: 0 calc(var(--gutter) / 2);
  grid-column: span var(--span); /* 关键代码：每个item跨越 --span 个网格 */
  box-sizing: border-box;
}

.col1 {
  --span: 1;
}
.col2 {
  --span: 2;
}
.col3 {
  --span: 3;
}
.col4 {
  --span: 4;
}
```

### 七. 两端对齐

### 八. 选择最佳的值

### 九. Logo 图标的对齐

### 十. 引用
[40 个 CSS 布局技巧（大漠）](https://mp.weixin.qq.com/s/6jZibuthus7l5PlhAbpHEQ)
