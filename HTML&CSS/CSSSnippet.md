## CSS 常用代码片段

#### background 背景简写
```css
.box {
    background: url('xxx') no-repeat left top / 100% auto;
}
```

#### - 文本溢出
```css
/* 单行文本溢出 */
.text-overflow {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}

/* 多行文本溢出 */
.text-overflow {
    position: relative;
    display: -webkit-box;
    overflow: hidden;
    text-overflow: ellipsis;
    word-break: break-all;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
}
```