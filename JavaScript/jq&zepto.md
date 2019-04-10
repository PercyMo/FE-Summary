



####1. 对比`append()`插入`<script>`标签情况

1. 问题场景：

    ```js
    $('body').append('<script src="js\/test.js"><\/script>')

    // test.js
    console.log('测试一下')
    ```
    以上js片段，当文档中引入jq时，可正常执行，而文档中如果使用的是zepto，则没任何反应（控制台中可见节点已正常插入）

2. 分析jq和zepto源码中对于append()的实现

    1. zepto
        zepto.js 里有一些关于DOM 节点插入操作的函数. 例如after, append, before, append 等, 它们都是由node.insertBefore来实现的

        在节点插入中还包含下面这段代码, 是用来处理插入script标签的情况
        ```js
        if (parentInDocument) traverseNode(node, function(el){ 
        if (el.nodeName != null && el.nodeName.toUpperCase() === 'SCRIPT' && 
        // type 为'' 或JavaScript , 不存在外部脚本 
            (!el.type || el.type === 'text/javascript') && !el.src){ 
            var target = el.ownerDocument ? el.ownerDocument.defaultView : window 
            target['eval'].call(target, el.innerHTML) 
        } 
        }) 
        ```
        之所以需要使用eval()来运行script标签里面的代码是因为,zepto将html片段转换为node节点是通过Element.innerHTML来实现的, 而通过innerHTML插入的script标签是不会执行的.
        ```js
        container = containers[name] 
        // 利用一个container 的innerHTML将 HTML片段转化为DOM结构 
        container.innerHTML = '' + html 
        // dom 为$.each(elements, callback) return elements,  
        // 即slice.call(container.childNodes)将childNodes转化为Array,以便$(Array)转化为Zepto对象 
        dom = $.each(slice.call(container.childNodes), function(){ 
        // 清空container 
        container.removeChild(this) 
        }) 
        ```

        上面代码中特殊处理的script是这种形式`"<script>console.log('测试代码')<\/script>"`  
        而并没有对`<script src="js\/test.js"><\/script>`做特殊处理，因此并不识别。

    2. jquery

        在jq中的domManip函数中，首先会通过
        ```js
        scripts = jQuery.map( getAll( fragment, "script" ), disableScript );
        ```
        解析出片段的完整属性，当发现是script标签并且有src时，会通过ajax方法访问这个地址
        ```js
        jQuery._evalUrl( node.src );

        jQuery._evalUrl = function( url ) {
            return jQuery.ajax({
                url: url,
                type: "GET",
                dataType: "script",
                async: false,
                global: false,
                "throws": true
            });
        };
        ```

        由此可见，在jq中是对这种情况做过处理的，所以可以正常使用。

3. 引用
    [zepto.js使用$.fn.append等方法插入script标签的情况](https://github.com/fangbinwei/zepto_src_analysis/issues/1)   
    [
    用jQuery的append方法插入script节点](https://segmentfault.com/q/1010000003696587)