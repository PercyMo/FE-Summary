## CSS经典布局

[Demo示例地址](http://me.vanilla.ink:3002/html&css/cssayout.html)

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

### 二. 等高布局

### 三. Sticky Footer

### 四. 均分列

### 五. 圣杯布局

### 六. 12列网格布局

### 七. 两端对齐

### 八. 选择最佳的值

### 九. Logo 图标的对齐

### 四. 引用
[40 个 CSS 布局技巧](https://mp.weixin.qq.com/s/6jZibuthus7l5PlhAbpHEQ)