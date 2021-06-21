## 代理模式

### 一. 介绍
#### 1. 定义
为其它对象提供一种代理以控制对这个对象的访问。（ES6中也新增了的Proxy功能）

#### 2. 适用场景
1. **远程代理**：为了一个对象在不同的地址空间提供局部代表，这样可以隐藏一个对象存在于不同地址空间的事实，就像 web service 里的代理类一样。（没太懂？Nginx反向代理？）
2. **虚拟代理**：根据需要创建开销很大的对象，它可以缓存实体的附加信息，以便延迟对它的访问，例如网站加载大图时，不能马上完成，使用虚拟代理缓存图片的大小信息，然后生成一张临时图片替代原图。
3. **安全代理**：用来控制真是对象访问时的权限，负责检查调用者是否具有实现一个请求所必须的访问权限。
4. **智能代理**：当调用真实的对象时，代理处理另外一些事情。例如C#里的垃圾回收，使用对象的时候会有引用计数，如果对象没有引用了，GC就可以回收它了。

#### 3. 优点
* 在不修改目标对象的前提下，能通过代理对象对目标功能扩展

#### 4. 缺点
1. 因为代理对象需要与目标对象实现一样的接口，所以会有很多代理类
2. 一旦接口增加方法，目标对象与代理对象都要维护
3. 增加代理类之后明显会拖慢处理时间

### 二. 实现
1. 虚拟代理实现图片 loading 效果  
    ```js
    // RealSubject
    const domImage = (function () {
      const imgEle = document.createElement("img");
      document.body.appendChild(imgEle);
      return {
        setSrc(src) {
          imgEle.src = src;
        },
      };
    })();

    // Proxy
    const ProxyImage = (function () {
      const img = new Image();
      img.onload = function () {
        domImage.setSrc(this.src); // 图片加载完设置真实src
      };
      return {
        setSrc(src) {
          domImage.setSrc("./loading.gif"); // 预先设置图片src为loading图
          img.src = src;
        },
      };
    })();

    ProxyImage.setSrc("./product.png");
    ```

2. 实现缓存
    ```js
    const mult = function () {
      const arr = Array.prototype.slice.call(arguments);
      console.log("Recalculate");
      return arr.reduce(
        (accumulator, currentValue) => accumulator * currentValue,
        1
      );
    };

    const ProxyMult = (function () {
      const cache = {};
      return function () {
        const args = Array.prototype.join.call(arguments, ",");
        if (args in cache) {
          console.log("Got result from cache");
          return cache[args]; // 使用缓存代理
        } else {
          return (cache[args] = mult.apply(this, arguments));
        }
      };
    })();

    ProxyMult(1, 2, 3, 4);
    // Recalculate
    // 24
    ProxyMult(1, 2, 3, 4);
    // Got result from cache
    // 24
    ```