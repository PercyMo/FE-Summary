## 无头浏览器 Puppeteer

`Puppeteer` 是一个由 `Google` 开发的 `Node.js` 库，它提供了一组API，可以通过Node.js控制 `Headless Chrome(无头浏览器)` 或者 `Chromium` 浏览器进行自动化操作。

### 一. 应用场景
* 对网页进行截图保存为图片或 pdf
* 做表单的自动提交、UI的自动化测试、模拟键盘输入等
* 爬虫：抓取单页应用(SPA)执行并渲染(解决传统 HTTP 爬虫抓取单页应用难以处理异步请求的问题)
* 网站监测：配合 `lighthouse` 或浏览器自带的一些调试工具和性能分析工具帮助我们分析问题，检查网站健康状况

### 二. 使用示例
创建浏览器示例，打开页面
```js
// 启动浏览器
const browser = await puppeteer.launch({
  // 关闭无头模式，方便我们看到这个无头浏览器执行的过程
  // headless: false,
});

// 打开空白页面
const page = await browser.newPage();

// 进行交互
// ...

// 关闭浏览器
await browser.close();
```

打开网址
```js
await page.goto('https://google.com/', {
  // 配置项
  // waitUntil: 'networkidle', // 等待网络状态为空闲的时候才继续执行
});
```

保存网页为 pdf
```js
await page.pdf({
  path: 'example.pdf',
  format: 'A4', // 保存尺寸
});
```