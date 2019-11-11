## doctype与文档模式

### 一. !DOCTYPE是什么
`!DOCTYPE`声明不是HTML标签，是一条指令，告诉浏览器页面是使用哪个 HTML 版本进行编写的。

在HTML4.01中，`<!DOCTYPE>` 声明需要引用 DTD，因为 HTML 4.01 基于 SGML。DTD 规定了标记语言的规则，这样浏览器才能正确地呈现内容。

HTML5 不基于 SGML，所以不需要引用 DTD。

### 二. 文档模式
#### 1. 文档模式的种类
1. 怪异模式  
    IE6789的是IE5.5的文档模式，IE10+和Chrome等浏览器是W3C规范的怪异模式
2. 标准模式（非怪异模式）  
    W3C标准的文档模式，但各浏览器的实现阶段不尽相同。
3. 准标准模式（有限怪异模式）  
    由于该模式离W3C标准仍然有一段距离，因此被称作准标准模式（或有限怪异模式）。IE6、7的标准模式实际上就是准标准模式，而IE8+才有实质上的标准模式。

#### 2. 不一样的标准模式和怪异模式
1. 虽说IE6、7、8、9、10、11均有**标准模式**，但由于W3C标准规范内容随时间的增改，而且浏览器对标准的实现是阶段性的，因此个版本的标准模式不尽相同。
2. IE6789的**怪异模式**其实就是IE5.5的文档模式，但从IE10开始它的遵守了W3C规范的怪异模式。

#### 3. 文档模式的影响
1. 布局  
    表格、单元格的样式等都受到文档模式的影响，尤其是盒子模型。
2. 解析  
    css和html的解析也会受到文档模式的影响，就像在IE678标准模式时，HTML解析时会将嵌套form下的子节点挪到上一节；而IE9标准模式时不会。
3. 脚本

### 三. `<!DOCTYPE>`与`<meta http-equiv="X-UA-Compatible">`共同影响文档模式
1. 所有IE浏览器在默认情况下（`<meta http-equiv="X-UA-Compatible">`与`<!DOCTYPE>`均没有），是采用怪异模式（Quirks）；
2. 当有`<!DOCTYPE>`时，均采用浏览器版本对应的标准模式（如IE8就采用IE8标准模式，IE11就采用IE11标准模式）。
3. **注意**：当出现`<meta http-equiv="X-UA-Compatible">`时，content属性值也会影响文档类型  
    `IE=5、IE=7、IE=EmulateIE7、IE=8、IE=EmulateIE8、IE=9、IE=10、 IE=11、 IE=Edge`
    1. IE=5：表示采用怪异模式；
    2. IE=7等纯数字的值：表示采用对应IE版本的标准模式，即使不是以`<!DOCTYPE>`作为文档第一行，文档模式依旧使用标准模式；
    3. IE=EmulateIE7等含EmulateIE字符串的值：表示采用模拟对应IE版本的模式，就是以`<!DOCTYPE>`作为文档第一行则采用标准模式，否则采用怪异模式。
    4. IE=Edge：表示采用浏览器自身版本的文档模式，如IE11，以`<!DOCTYPE html>`作为文档第一行则采用IE11标准模式，否则采用怪异模式。

### 四. 附：常用的DOCTYPE声明
* HTML 5  
    ```html
    <!DOCTYPE html>
    ```

* HTML 4.01 Strict  
    该 DTD 包含所有 HTML 元素和属性，但不包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。
    ```html
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
    ```

* HTML 4.01 Transitional  
    该 DTD 包含所有 HTML 元素和属性，包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。
    ```html
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    ```

* HTML 4.01 Frameset  
    该 DTD 等同于 HTML 4.01 Transitional，但允许框架集内容。
    ```html
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">
    ```

* XHTML 1.0 Strict  
    该 DTD 包含所有 HTML 元素和属性，但不包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。必须以格式正确的 XML 来编写标记。
    ```html
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
    ```

* XHTML 1.0 Transitional  
    该 DTD 包含所有 HTML 元素和属性，包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。必须以格式正确的 XML 来编写标记。
    ```html
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    ```

* XHTML 1.0 Frameset  
    该 DTD 等同于 XHTML 1.0 Transitional，但允许框架集内容。
    ```html
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">
    ```

* XHTML 1.1  
    该 DTD 等同于 XHTML 1.0 Strict，但允许添加模型（例如提供对东亚语系的 ruby 支持）。
    ```html
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
    ```

### 五. 引用
[HTML <!DOCTYPE> 标签](https://www.w3school.com.cn/tags/tag_doctype.asp)

[JS魔法堂：浏览器模式和文档模式怎么玩？](https://www.cnblogs.com/fsjohnhuang/p/3817418.html)