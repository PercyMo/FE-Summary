## JavaScript 手撕代码

### 手动实现 call、apply、bind
1. call
```js
Function.prototype.myCall = function(context, ...args) {
  if (this === Function.prototype) {
    return; // 用于防止 Function.prototype.myCall() 直接调用
  }
  context = context || window;
  const fn = Symbol();
  context[fn] = this;
  const reslut = context[fn](...args);
  delete context[fn];
  return reslut;
};
```

2. apply
```js
Function.prototype.myApply = function(context, args) {
  if (this === Function.prototype) {
    return;
  }
  context = context || window;
  const fn = Symbol();
  context[fn] = this;
  const result = Array.isArray(args) ? context[fn](...args) : context[fn]();
  delete context[fn];
  return result;
};
```

3. bind
```js
Function.prototype.myBind = function(context, ...args) {
  if (this === Function.prototype) {
    throw new TypeError('Error');
  }
  const self = this;
  return function F(...newArgs) {
    // 判断是否用于构造函数
    if (this instanceof F) {
      return new self(...args, ...newArgs);
    }
    return self.apply(context, args.concat(newArgs));
  };
};
```

### 防抖节流
**防抖**
```js
const debounce = function(fn, wait = 0, immediate) {
  let timer = null;
  return function(...args) {
    clearTimeout(timer);
    if (immediate && !timer) {
      fn.apply(this, args);
    }
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, wait);
  };
};
```
**节流**
```js
// 时间戳实现：第一次一定触发，最后一次不一定触发
const throttle = function(fn, wait) {
  let pre = 0;
  return function(...args) {
    const date = new Date();
    if (date - pre >= wait) {
      pre = date;
      return fn.apply(this, args);
    }
  };
};

// 定时器实现：第一次一定触发，最后一次不一定触发
const throttle = function(fn, wait) {
  let timer = null;
  return function(...args) {
    if (!timer) {
      timer = setTimeout(() => {
        fn.apply(this, args);
        clearTimeout(timer);
        timer = null;
      }, wait);
    }
  };
};

// 结合定时器与时间戳：第一次和最后一次都会触发
const throttle = function(fn, wait) {
  let timer = null;
  let pre = 0;
  return function(...args) {
    clearTimeout(timer);
    const date = new Date();

    if (date - pre >= wait) {
      pre = date;
      fn.apply(this, args);
    } else {
      timer = setTimeout(() => {
        fn.apply(this, args);
      }, wait);
    }
  };
};
```

### 深浅拷贝
```js
// 可继续遍历的数据类型
const setTag = '[object Set]';
const mapTag = '[object Map]';
const arrayTag = '[object Array]';
const objectTag = '[object Object]';
const argsTag = '[object Arguments]';

// 不可继续遍历的数据类型
const boolTag = '[object Boolean]';
const dateTag = '[object Date]';
const errorTag = '[object Error]';
const numberTag = '[object Number]';
const regexpTag = '[object RegExp]';
const stringTag = '[object String]';
const symbolTag = '[object Symbol]';
const funcTag = '[object Function]';

const deepTag = [setTag, mapTag, arrayTag, objectTag, argsTag];

// 使用while遍历，提升遍历性能
function forEach(array, iteratee) {
  let i = -1;
  const len = array.length;
  while (++i < len) {
    iteratee(array[i], i);
  }
}

// 判断是否为引用类型
function isObject(target) {
  const type = typeof target;
  return target !== null && (type === 'object' || type === 'funciton');
}

// 获取数据实际类型
function getType(target) {
  return Object.prototype.toString.call(target);
}

// 初始化被克隆的对象，同时保持原型指向
// target为对象时，返回{}；target为数组时，返回[]...
function getInit(target) {
  const Ctor = target.constructor;
  return new Ctor;
}

// 克隆正则表达式
function cloneReg(target) {
  // 从正则末尾截取修饰符
  const reFlags = /\w*$/;
  const result = new target.constructor(target.source, reFlags.exec(target));
  // RegExp对象是有状态的，克隆上一次匹配成功的位置
  result.lastIndex = target.lastIndex;
  return result;
}

// 克隆symbol类型
function cloneSymbol(target) {
  return Object(Symbol.prototype.valueOf.call(target));
}

// 克隆函数
function cloneFunction(taget) {
  // 蚂蚁金服一位哥们骚气的写法，目前没发现什么问题
  return eval('(' + target.toString() + ')');
}

// 克隆不可遍历类型
function cloneOtherType(target, type) {
  const Ctor = target.constructor;
  switch (type) {
    case boolTag:
    case numberTag:
    case stringTag:
    case errorTag:
    case dateTag:
      return new Ctor(target);
    case regexpTag:
      return cloneReg(target);
    case symbolTag:
      return cloneSymbol(target);
    case funcTag:
      return cloneFunction(target);
    default:
      return null;
  }
}

// 使用WeakMap保存当前对象和拷贝对象关系，及时释放内存
function clone(target, wm = new WeakMap()) {
  // 克隆原始类型 排除了object(不包括null)和function
  if (!isObject(target)) {
    return target;
  }

  // 初始化
  const type = getType(target);
  let cloneTarget;
  if (deepTag.includes(type)) {
    cloneTarget = getInit(target);
  } else {
    return cloneOtherType(target, type);
  }

  // 防止循环引用
  const stacked = wm.get(target);
  if (stacked) {
    return stacked;
  }
  wm.set(target, cloneTarget);

  // 克隆set
  if (type === setTag) {
    target.forEach((value) => {
      cloneTarget.add(clone(value, wm));
    })
    return cloneTarget;
  }

  // 克隆map
  if (type === mapTag) {
    target.forEach((value, key) => {
      cloneTarget.set(key, clone(value, wm));
    })
    return cloneTarget;
  }

  // 克隆对象和数组
  const keys = type === arrayTag ? undefined : Object.keys(target);
  forEach(keys || target, (value, key) => {
    if (keys) {
      key = value;
    }
    cloneTarget[key] = clone(target[key], wm);
  })

  return cloneTarget;
};
```

### 数组去重
**Object**
```js
// 开辟一个外部空间标记元素是否出现过
const unique = (array) => {
  const obj = {};
  return array.filter((item) => obj[item] ? false : (obj[item] = true));
};
```
**indexOf + filter**
```js
const unique = (array) => {
  return array.filter((el, i) => array.indexOf(el) === i);
};
```
**Set**
```js
const unique = (array) => [...new Set(array)];
const unique = (array) => Array.from(new Set(array));
```
**先排序后去重**
```js
const unique = (array) => {
  array.sort((a, b) => a - b);
  return array.filter((item, index) => item !== array[index - 1]);
};
```
**去除重复的值**
```js
// 不同于上面的去重，这里只要是重复的值，直接移除
const unique = (array) => {
  return array.filter((el) => array.indexOf(el) === array.lastIndexOf(el));
};
```

### 数组扁平化
```js
// 基础实现
const flatten = (array) => {
  let result = [];
  array.forEach((item) => {
    if (Array.isArray(item)) {
      result = result.concat(flatten(item));
    } else {
      result.push(item);
    }
  })
  return result;
};

// 使用 reduce 简化
const flatten = (array) => {
  return array.reduce(
    (pre, current) =>
      Array.isArray(current) ?
        pre.concat(flatten(current)) :
        pre.concat(current)
    , []);
};

// 指定深度扁平数组
const flatten = (array, deep = 1) => {
  return array.reduce(
    (pre, current) => {
      return Array.isArray(current) && deep > 1 ?
        pre.concat(flatten(current, (deep - 1))) :
        pre.concat(current);
    }
    , []);
};
```

### 数组最值
```js
// 使用 reduce
const returnMax = (array) => {
  return array.reduce((pre, current) => pre > current ? pre : current);
};

// 使用 Math.max
// Math.max 的参数原本是一组数字，只要让它接受数组即可
Math.max(...array);
Math.max.apply(null, array);
```

### 模拟实现数组方法
**模拟实现 Array.map**
```js
Array.prototype.myMap = function(fn) {
  return this.reduce((pre, current, index) => {
    return pre.concat(fn.call(this, current, index, this));
  }, []);
};
```
**模拟实现 Array.filter**
```js
Array.prototype.myFilter = function(fn) {
  return this.reduce(
    (pre, current, index) => 
      fn.call(this, current, index, this) ?
        pre.concat(current) :
        pre
  , []);
};
```

### 数组乱序-洗牌算法
```js
function shuffle(array) {
  const _arr = array.slice();
  for (let i = _arr.length - 1; i > 0; i--) {
    const random = Math.floor(Math.random() * i);
    [_arr[i], _arr[random]] = [_arr[random], _arr[i]];
  }
  return _arr;
}
```

### 函数柯里化
```js
function curry(fn, ...args) {
  if (args.length >= fn.length) {
    return fn(...args);
  } else {
    return (...newArgs) => currying(fn, ...args, ...newArgs);
  }
}
```

### 手动实现 JSONP
```js
let count = 0;

function generateUrl(url, params) {
  let dataString = url.indexOf('?') > -1 ? '' : '?';
  for (let key in params) {
    dataString += `${key}=${params[key]}&`;
  }
  return url + dataString;
}

function noop() {};

function jsonp(url, data = {}) {
  return new Promise((resolve, reject) => {
    const id = `__jp${count++}`;
    let scriptEl;

    data.jsonpCallback = id;

    function cleanup() {
      window[id] = noop;
      document.body.removeChild(scriptEl);
    }
    
    // 挂载回调函数
    window[id] = function(data) {
      cleanup();
      resolve(data);
    }
    
    // 创建 script 标签，发送请求
    scriptEl = document.createElement('script');
    scriptEl.src = generateUrl(url, data);
    scriptEl.onerror = (err) => {
      cleanup();
      reject(err);
    };
    document.body.appendChild(scriptEl);
  });
};
```

### Promise 模拟实现
```js
const isFunction = (variable) => typeof variable === 'function';

const PENDING = 'pending'; // 进行中
const FULFILLED = 'fulfilled'; // 已成功
const REJECTED = 'rejected'; // 已失败

class MyPromise {
  constructor(fn) {
    // MyPromise类接受一个函数作为参数
    if (!isFunction(fn)) {
      throw new Error('MyPromise must accept a function as a parameter');
    }
    // 开始状态 pending
    this._status = PENDING;
    // 返回值
    this._value = undefined;
    // 成功回调函数队列
    this._fulfilledQueues = [];
    // 失败回调函数队列
    this._rejectedQueues = [];

    // 执行 fn 函数
    try {
      fn(this._resolve.bind(this), this._reject.bind(this));
    } catch (err) {
      this._reject(err);
    }
  }
  _resolve(val) {
    if (this._status !== PENDING) return;
    const run = () => {
      // 依次执行成功队列中的函数，并清空队列
      const runFulfilled = (value) => {
        let cb;
        while ((cb = this._fulfilledQueues.shift())) {
          cb(value);
        }
      };

      // 依次执行失败队列中的函数，并清空队列
      const runRejected = (error) => {
        let cb;
        while ((cb = this._rejectedQueues.shift())) {
          cb(error);
        }
      };

      // 如果resolve的参数为Promise对象，则必须等待Promise对象状态改变后，
      // 当前Promise对象的状态才会改变，且状态取决于参数Promise对象的状态
      if (val instanceof MyPromise) {
        val.then(
          (value) => {
            this._status = FULFILLED;
            this._value = value;
            runFulfilled(value);
          },
          (error) => {
            this._status = REJECTED;
            this._value = error;
            runRejected(error);
          },
        );
      } else {
        this._status = FULFILLED;
        this._value = val;
        runFulfilled(val);
      }
    };
    // 为了支持同步的Promise，这里采用异步调用
    setTimeout(run, 0);
  }
  _reject(err) {
    if (this._status !== PENDING) return;
    const run = () => {
      this._status = REJECTED;
      this._value = err;
      // 依次执行成功队列中的函数，并清空队列
      let cb;
      while ((cb = this._rejectedQueues.shift())) {
        cb(err);
      }
    };
    // 为了支持同步的Promise，这里采用异步调用
    setTimeout(run, 0);
  }
  then(onFulfilled, onRejected) {
    const { _value, _status } = this;
    // 返回一个新的 Promise 对象
    return new MyPromise((resolveNext, rejectNext) => {
      // 封装一个成功时执行的函数
      const fulfilled = (value) => {
        try {
          if (!isFunction(onFulfilled)) {
            // 值穿透发生在此处，相当于then方法执行时并未对_value进行修改
            resolveNext(value);
          } else {
            const res = onFulfilled(value);
            if (res instanceof MyPromise) {
              // 如果当前回调函数返回Promise对象，必须等待其状态改变后再执行下一回调
              res.then(resolveNext, rejectNext);
            } else {
              // 否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              resolveNext(res);
            }
          }
        } catch (err) {
          // 如果新的函数执行出错，新的Promise对象状态为失败
          rejectNext(err);
        }
      };

      // 封装一个失败时执行的函数
      const rejected = (error) => {
        try {
          if (!isFunction(onRejected)) {
            // 如果失败回调不是函数，会调用当前函数的reject方法
            rejectNext(error);
          } else {
            const res = onRejected(error);
            if (res instanceof MyPromise) {
              // 如果当前回调函数返回Promise对象，必须等待其状态改变后再执行下一回调
              res.then(resolveNext, rejectNext);
            } else {
              // 否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              resolveNext(res);
            }
          }
        } catch (err) {
          rejectNext(err);
        }
      };

      switch (_status) {
        case PENDING:
          this._fulfilledQueues.push(fulfilled);
          this._rejectedQueues.push(rejected);
          break;
        case FULFILLED:
          /**
           * p.then 时，promise 状态已经变了，比如:
           * const p = Promise.resolve(1);
           * setTimeout(() => { p.then((val) => console.log(val))}, 1000);
           */
          fulfilled(_value);
          break;
        case REJECTED:
          rejected(_value);
          break;
      }
    });
  }
  catch(onRejected) {
    // 实际相当于返回一个只有失败回调的then方法
    return this.then(undefined, onRejected);
  }
  // 添加静态方法 resolve
  static resolve(value) {
    // 如果参数是Promise实例，直接返回这个实例
    if (value instanceof MyPromise) return value;
    return new MyPromise((resolve) => resolve(value));
  }
  // 添加静态方法 reject
  static reject(value) {
    return new MyPromise((resolve, reject) => reject(value));
  }
  // 添加静态方法 all
  static all(promises) {
    // 返回一个新的Promise实例
    return new MyPromise((resolve, reject) => {
      // 返回有序的值的集合
      const values = [];
      let count = 0;
      for (const [i, p] of promises.entries()) {
        // 数组参数如果不是Promise实例，先调用Promise.resolve
        this.resolve(p).then(
          (val) => {
            values[i] = val;
            count++;
            // 所有状态都变成fulfilled时，返回的Proimse状态就变成fulfilled
            if (count === promises.length) resolve(values);
          },
          (err) => {
            // 只要有一个状态变为rejected，返回的Promise状态就变成rejected
            reject(err);
          },
        );
      }
    });
  }
  // 添加静态方法 race
  static race(promises) {
    return new MyPromise((resolve, reject) => {
      for (const p of promises) {
        // 只要有一个实例率先改变状态，返回的Promise状态就跟着改变
        this.resolve(p).then(
          (val) => {
            resolve(val);
          },
          (err) => {
            reject(err);
          },
        );
      }
    });
  }
  // 添加静态方法 finally
  finally(cb) {
    // 通catch方法一样，实际返回了一个then方法
    return this.then(
      // 之所以再包装一层Promise.resolve，是为了保证返回值不变
      // Promise.resolve(3).finally(() => {}) // 3
      (val) => MyPromise.resolve(cb()).then(() => val),
      (err) =>
        MyPromise.resolve(cb()).then(() => {
          throw err;
        }),
    );
  }
}
```

### 手动实现 ES5 继承
```js
function People() {
  this.type = 'prople'
}

People.prototype.eat = function () {
  console.log('吃东西啦');
}

function Man(name) {
  this.name = name;
  this.color = 'black';
}

// ...

// 使用方式
const man = new Man();
```
1. 原型继承
    ```js
    Man.prototype = new People();
    ```
2. 构造继承
    ```js
    function Man(name) {
      People.call(this);
      this.name = name;
      this.color = 'black';
    }
    ```
3. 组合继承
    ```js
    function Man(name) {
      People.call(this);
      this.name = name;
      this.color = 'black';
    }
    Man.prototype = People.prototype;
    ```
4. 寄生继承
    ```js
    function Man(name) {
      People.call(this);
      this.name = name;
      this.color = 'black';
    }
    Man.prototype = Object.create(People.prototype, {
      constructor: {
        value: Man
      }
    });
    ```
5. inherits 函数
    ```js
    function inherits(ctor, superCtor) {
      ctor.super_ = superCtor;
      ctor.prototype = Object.create(superCtor.prototype, {
        constructor: {
          value: ctor,
          enumerable: false,
          writable: true,
          configurable: true
        }
      });
    };
    function Man(name) {
      People.call(this);
      this.name = name;
      this.color = 'black';
    }
    inherits(Man, People);
    ```

### instanceof 模拟实现
```js
const oInstanceof = function (ins, cons) {
  if (!ins) return false;

  return ins.__proto__ === cons.prototype
    ? true
    : oInstanceof(ins.__proto__, cons);
};
```

### - 基于 promise 的 ajax 封装
```js
function getStringParams(params) {
  let str = '';
  for (let key in params) {
    str += `${key}=${params[key]}&`;
  }
  return str;
}

function ajax(url = '', method = 'get', params = {}) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();

    // 拼接 get 请求 url
    const stringParams = getStringParams(params);
    if (method === 'get' && stringParams) {
      url += url.indexOf('?') > -1 ? stringParams : `?${stringParams}`;
    }

    xhr.open(method, url, true);
    xhr.onload = function() {
      const result = {
        status: xhr.status,
        statusText: xhr.statusText,
        headers: xhr.getAllResponseHeaders(),
        data: xhr.response || xhr.responseText,
      };
      if (xhr.status >= 200 && xhr.status < 300 || xhr.status === 304) {
        resolve(result);
      } else {
        reject(result);
      }
    };

    // 请求头
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    xhr.withCredentials = true;

    // 异常处理
    xhr.onerror = function() {
      reject(new TypeError('请求出错'));
    };
    xhr.timeout = function() {
      reject(new TypeError('请求超时'));
    };
    xhr.onabort = function() {
      reject(new TypeError('请求被终止'));
    };
    
    if (method === 'post') {
      xhr.send(stringParams);
    } else {
      xhr.send();
    }
  });
}
```

### 异步循环打印
```js
const sleep = (i) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(i);
    }, 1000);
  });
};

async function start() {
  // 和 forEach 写法区分，forEach 里的等待是不生效的
  for (let i = 0; i < 3; i++) {
    const result = await sleep(i);
    console.log(result);
  }
}

start();
```

### 图片懒加载
1. 监听页面滚动  
    ```js
    import { throttle } from 'lodash-es';

    const imgs = document.querySelectorAll('img');
    const windowHeigh = window.innerHeight;
    let count = 0;
    const lazyLoad = () => {
      if (count >= imgs.length) return;
      const scrollTop = document.documentElement.scrollTop;
      for (let i = count; i < imgs.length; i++) {
        const item = imgs[i];
        const isInViewport = (item.offsetTop - scrollTop) < windowHeigh;
        if (isInViewport && !item.getAttribute('lazy')) {
          item.setAttribute('src', item.getAttribute('data-src'));
          item.setAttribute('lazy', 'loaded');
          count++;
        }
      }
    };

    window.addEventListener('scroll', throttle(lazyLoad, 150));
    ```

2. 基于 `Intersection Observer API` 实现相交监听  
    ```js
    if (IntersectionObserver) {
      const imgs = document.querySelectorAll('img');
      
      const observer = new IntersectionObserver((entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            const target = entry.target;
            target.setAttribute('src', target.getAttribute('data-src'));
            observer.unobserve(target);
          }
        });
      });
      
      for (let i = 0; i < imgs.length; i++) {
        observer.observe(imgs[i]);
      }
    }
    ```
