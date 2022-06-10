## 模板引擎原理

### 一. 实现原理
```js
var Template = function(tpl) {
  var 
    fn,
    match,
    code = ['var r = [];'],
    re = /<%\s*([^%>]+)\s*%>/g,
    reExp = /(^()?(if|else|for|switch|case|break|{|}))(.*)?/g,
    cursor = 0;

  var addLine = function(text, js) {
    js ? (text.match(reExp) ? code.push(text) : code.push(`r.push(${text});`)) :
      (text !== '' && code.push(`r.push(\'${text.replace(/\n/g, '\\n')}\');`))
    return addLine;
  };

  while (match = re.exec(tpl)) {
    addLine(tpl.slice(cursor, match.index))(match[1], true);
    cursor = match.index + match[0].length;
  }
  addLine(tpl.slice(cursor));
  code.push('return r.join("");');

  fn = new Function(code.join(''));
  
  this.render = function(model) {
    return fn.apply(model);
  };
};
```
使用
```js
var tpl = new Template(`
  <% for (var key in this.list) { %>
    <p>Item: <% this.list[key] %></p>
    <p>Today: <% this.date %></p>
    <a href="<% this.user.id %>">
      <% this.user.company %>
    </a>
  <% } %>`
);

var html = tpl.render({
  list: ['Vue', 'React'],
  date: 20150316,
  user: {
    id: 'A-000&001',
    company: 'AT&T'
  }
});

// 输出 html
<p>Item: Vue</p>
<p>Today: 20150316</p>
<a href="A-000&001">
  AT&T
</a>

<p>Item: React</p>
<p>Today: 20150316</p>
<a href="A-000&001">
  AT&T
</a>
```

### 二. 引用
[JavaScript模板引擎原理，几行代码的事儿](https://www.cnblogs.com/hustskyking/p/principle-of-javascript-template.html)  

[JavaScript template engine in just 20 lines](https://krasimirtsonev.com/blog/article/Javascript-template-engine-in-just-20-line)  