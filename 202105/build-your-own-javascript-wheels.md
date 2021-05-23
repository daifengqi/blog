# 手动造轮子之JavaScript特性实现

# Build Your Own JavaScript Wheels

## 2021 年 5 月 22 日

## May 22 2021



### 参考

- [自检清单](http://www.conardli.top/blog/article/%E7%BB%BC%E5%90%88/%E3%80%90%E8%87%AA%E6%A3%80%E3%80%91%E5%89%8D%E7%AB%AF%E7%9F%A5%E8%AF%86%E6%B8%85%E5%8D%95.html#%E4%B8%80%E3%80%81javascript%E5%9F%BA%E7%A1%80)
- [淘系消费者运营前端团队](https://github.com/mqyqingfeng/Blog)
- [Windy的文章](https://mp.weixin.qq.com/s/DgCCWyFujBImxjqC3vTwcA)



### 前言

废话少说，放码过来。

### 手动实现一个instanceof

`left instanceof right`作用是判断一个实例`left`是否由指定的构造函数`right`创建。

因此，只需要在`left`的原型链上查找，是否能追溯到`right`的原型即可，道理很简单。

```javascript
function instanceOf(left, right) {
  const rightPrototype = right.prototype; // 取右表达式的 prototype 值
  let leftProto = left.__proto__; // 取左表达式的__proto__值
  while (true) {
    if (leftProto === null) {
      return false;
    }
    if (leftProto === rightPrototype) {
      return true;
    }
    leftProto = leftProto.__proto__;
  }
}

```

### 手动实现一个new

> new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象类型之一。
>
> ——MDN

我们随便试验一下就知道，`new`运算符创造的实例可以做两件事：

- 访问到构造函数里的属性
- 访问到构造函数.prototype 中的属性

具体地，`new`操作符会执行如下流程：

1. 用new Object() 的方式新建了一个对象 obj；
2. 取出第一个参数，就是我们要传入的构造函数；
3. （🌟）将 obj 的原型指向构造函数，这样 obj 就可以访问到构造函数原型中的属性；
4. （🌟）使用 apply，改变构造函数 this 的指向到新建的对象，这样 obj 就可以访问到构造函数中的属性；
5. 返回 obj

因为`new`是一个关键字，使用时要用到一个构造函数加上构造函数的参数，我们无法创建一个关键字，故模拟一个函数如下，

```javascript
// 不允许es6方法
function newES5() {
  // 创建一个新对象
  var obj = new Object(),
  // 取出函数的第一个参数，即构造函数
  var Constructor = [].shift.call(arguments);
  // 修改原型指向
  obj.__proto__ = Constructor.prototype;
  // 给定this指向obj的情况下，执行构造函数
  var ret = Constructor.apply(obj, arguments);
  // 处理构造函数有返回值的特殊情况，如果返回值是原始类型，则无视构造函数的返回值，直接返回obj
  // 否则返回ret
  return typeof ret === 'object' ? ret : obj;
}
```

在ES6以后，通过扩展运算符，Object的新方法，有更简洁的方法，

```javascript
// 允许es6方法
function newES6(Fn, ...args) {
    // 1. 创建一个继承构造函数.prototype的空对象
    const obj = Object.create(Fn.prototype);
    // 2. 让空对象作为函数 A 的上下文，并调用 A
    const ret = Fn.call(obj, ...args);
    // 3. 如果构造函数返回一个对象，那么直接 return 它，否则返回内部创建的新对象
    return ret instanceof Object ? ret : obj;
}
```

实际上，ES6提供了不使用new来调用构造函数的方法：`Reflect.construct`，详情见[这篇文章](https://cloud.tencent.com/developer/article/1526901)。

### 手动实现call和apply

> call() 方法在使用一个指定的 this 值和若干个指定的参数值的前提下调用某个函数或方法。
>
> ——MDN

`call()`方法是`Function.prototype`，即函数原型里的方法，必须是一个函数才能调用，它为函数的运行提供了上下文，即`this`指向。

脑洞一开，我们模拟一个`call`方法的步骤可以分为：

1. 将函数设为对象的属性
2. 执行该函数
3. 删除该函数

代码如下，

```javascript
Function.prototype.callES3 = function (context) {
  if (this === Function.prototype) {
    return undefined; // 用于防止 Function.prototype.myCall() 直接调用
  }
  // call方法允许传入null作为上下文，在浏览器环境下会自动转换为window
  var context = context || window;
  context.fn = this;
  // 获取执行函数的参数
  var args = [];
  for (var i = 1, len = arguments.length; i < len; i++) {
    args.push("arguments[" + i + "]");
  }
  // 获取执行结果
  var result = eval("context.fn(" + args + ")");
  // 删除上下文里的函数
  delete context.fn;
  return result;
};

```

`apply`方法与`call`几乎一致，只是执行函数的参数单通过一个数组传递进入，而不是像`call`一样不定长传参，这里就不再写了。

一个ES6的写法，

```javascript
Function.prototype.callES6 = function (context = window, ...args) {
    if (this === Function.prototype) {
      return undefined; // 用于防止 Function.prototype.myCall() 直接调用
    }
    const fn = Symbol();
    context[fn] = this;
    const result = context[fn](...args);
    delete context[fn];
    return result;
  };
};

```

这里进行了一个优化，赋值`fn = Symbol()`，这样可以避免重名。

### 手动实现bind

> `bind()` 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。
>
> ——MDN

实现如下，

```javascript
Function.prototype.bindES6 = function (context, ...args1) {
  if (this === Function.prototype) {
    throw new TypeError("Error");
  }
  const _this = this;
  return function F(...args2) {
    // 判断是否用于构造函数
    if (this instanceof F) {
      return new _this(...args1, ...args2);
    }
    // 否则返回...
    return _this.apply(context, args1.concat(args2));
  };
};
```

### 手写Object.create

> **`Object.create()`**方法创建一个新对象，使用现有对象来提供新创建的对象的__proto__。

MDN提供的Polyfill，

```javascript
if (typeof Object.create !== "function") {
    Object.create = function (proto, propertiesObject) {
        if (typeof proto !== 'object' && typeof proto !== 'function') {
            throw new TypeError('Object prototype may only be an Object: ' + proto);
        } else if (proto === null) {
            throw new Error("This browser's implementation of Object.create is a shim and doesn't support 'null' as the first argument.");
        }

        if (typeof propertiesObject !== 'undefined') throw new Error("This browser's implementation of Object.create is a shim and doesn't support a second argument.");

        function F() {}
        F.prototype = proto;

        return new F();
    };
}
```

### 手写Object.assign

> `Object.assign(target, ...sources)` 方法用于将所有可枚举属性的值从一个或多个源对象分配到目标对象。它将返回目标对象。

`Object.assign` 本质是使用源对象的`[[Get]]`和目标对象的`[[Set]]`，所以它会调用相关 getter 和 setter。因此，如果想要拷贝getter函数，就不能使用`Object.assign` ，应该使用`Object.getOwnPropertyDescriptor`以及`Object.defineProperty`。

注意，`Object.assign`也会拷贝Symbol，即使它是不可枚举的。

MDN官方提供polyfill，它不支持 symbol 属性，是因为 ES5 中本来就不存在 symbols ，

```javascript
if (typeof Object.assign !== 'function') {
  // Must be writable: true, enumerable: false, configurable: true
  Object.defineProperty(Object, "assign", {
    value: function assign(target, varArgs) { // .length of function is 2
      'use strict';
      if (target === null || target === undefined) {
        throw new TypeError('Cannot convert undefined or null to object');
      }

      var to = Object(target);
			
      // get the value of every source
      for (var index = 1; index < arguments.length; index++) {
        var nextSource = arguments[index];

        if (nextSource !== null && nextSource !== undefined) {
          for (var nextKey in nextSource) {
            // Avoid bugs when hasOwnProperty is shadowed
            if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
              to[nextKey] = nextSource[nextKey];
            }
          }
        }
      }
      return to;
    },
    writable: true,
    configurable: true
  });
}
```

### Ajax

使用`XMLHttpRequest`实现，

```javascript
function ajax(url,method,body,headers){
    return new Promise((resolve,reject)=>{
        let req = new XMLHttpRequest();
        req.open(methods,url);
        for(let key in headers){
            req.setRequestHeader(key,headers[key])
        }
        req.onreadystatechange(()=>{
            if(req.readystate == 4){
                if(req.status >= '200' && req.status <= 300){
                    resolve(req.responeText)
                }else{
                    reject(req)
                }
            }
        })
        req.send(body)
    })
}
```

新的名为`fetch`的Web API本身就是基于Promise实现的，可以直接使用，并且是一个更加简洁的API。

>[MDN Useing fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)

```javascript
const data = { username: 'example' };

fetch('https://example.com/profile', {
  method: 'POST', // or 'PUT'
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify(data),
})
.then(response => response.json())
.then(data => {
  console.log('Success:', data);
})
.catch((error) => {
  console.error('Error:', error);
});
```

`fetch`有一些常用的场合，比如发送post请求以上传JSON（如上），或者通过`PUT`更新文件。

### 防抖（debounce） & 节流（throttle)

**防抖：在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。**

```javascript
let debounce = (fn,time = 1000) => {
    let timeLock = null;

    return function (...args){
        clearTimeout(timeLock);
        timeLock = setTimeout(()=>{
            fn(...args)
        },time);
    }
}
```

**节流：一定时间内只触发一次函数，没有重新计时。**

```javascript
let throttle = (fn,time = 1000) => {
    let flag = true;

    return function (...args){
        if(flag){
            flag = false;
            setTimeout(()=>{
                flag = true;
                fn(...args)
            },time)
        }
    }
}
```

### 深拷贝

某一种写法，供参考；

```javascript
function deepClone(obj, hash = new WeakMap()){
    if(obj instanceof RegExp) return new RegExp(obj);
    if(obj instanceof Date) return new Date(obj);
    if(obj === null || typeof obj !== 'object') return obj;
    //循环引用的情况
    if(hash.has(obj)){
        return hash.get(obj)
    }
    //new 一个相应的对象
    //obj为Array，相当于new Array()
    //obj为Object，相当于new Object()
    let constr = new obj.constructor();
    hash.set(obj,constr);
    for(let key in obj){
        if(obj.hasOwnProperty(key)){
            constr[key] = deepClone(obj[key],hash)
        }
    }
    //考虑symbol的情况
    let symbolObj = Object.getOwnPropertySymbols(obj)
    for(let i=0;i<symbolObj.length;i++){
        if(obj.hasOwnProperty(symbolObj[i])){
            constr[symbolObj[i]] = deepClone(obj[symbolObj[i]],hash)
        }
    }
    return constr
}
```

### 数组扁平化（递归写法、循环写法）

使用ES6的递归的写法，

```javascript
function fnES6(arr){
   return arr.reduce((prev,cur)=>{
      return prev.concat(Array.isArray(cur)?fn(cur):cur)
   },[])
}
```

非递归写法，利用闭包，

```javascript
var arr = [1, 2, 3, [4, 5], [6, [7, [8]]]];
/** * 使用递归的方式处理 * wrap 内保
存结果 ret * 返回一个递归函数 **/
function wrap() {
  var ret = [];
  return function flat(a) {
    for (var item of a) {
      if (item.constructor === Array) {
        ret.concat(flat(item));
      } else {
        ret.push(item);
      }
    }
    return ret;
  };
}
console.log(wrap()(arr));
```

### 柯里化

略，参考JavaScript函数专题。

### 每隔一秒打印1，2，3，4

```javascript
for (var i=1; i<=5; i++) {
  (function (i) {
    setTimeout(() => console.log(i), 1000*i)
  })(i)
}
```

> 思考题：直接在循环体内执行setTimeout函数，会打印什么。

### 手写Node核心模块EventEmitter

略。

### 手写sleep

```javascript
function sleep(delay) {
  var start = new Date().getTime();
  while (new Date().getTime() - start < delay) {
    continue;
  }
}
```

### 正则表达式：解析URL Params

我们想实现一个`parseParam`函数，具有功能，

```javascript
let url = 'http://www.domain.com/?user=anonymous&id=123&id=456&city=%E5%8C%97%E4%BA%AC&enabled';
parseParam(url)
/* 结果
{ user: 'anonymous',
  id: [ 123, 456 ], // 重复出现的 key 要组装成数组，能被转成数字的就转成数字类型
  city: '北京', // 中文需解码
  enabled: true, // 未指定值得 key 约定为 true
}
*/
```

实现如下，

```javascript
function parseParam(url) {
  const paramsStr = /.+\?(.+)$/.exec(url)[1]; // 将 ? 后面的字符串取出来
  const paramsArr = paramsStr.split('&'); // 将字符串以 & 分割后存到数组中
  let paramsObj = {};
  // 将 params 存到对象中
  paramsArr.forEach(param => {
    if (/=/.test(param)) { // 处理有 value 的参数
      let [key, val] = param.split('='); // 分割 key 和 value
      val = decodeURIComponent(val); // 解码
      val = /^\d+$/.test(val) ? parseFloat(val) : val; // 判断是否转为数字

      if (paramsObj.hasOwnProperty(key)) { // 如果对象有 key，则添加一个值
        paramsObj[key] = [].concat(paramsObj[key], val);
      } else { // 如果对象没有这个 key，创建 key 并设置值
        paramsObj[key] = val;
      }
    } else { // 处理没有 value 的参数
      paramsObj[param] = true;
    }
  })

  return paramsObj;
}
```

### 手写模板字符串（ES6模板字面量）引擎

> “模板字面量”是允许嵌入表达式的字符串字面量。你可以使用多行字符串和字符串插值功能。它们在ES2015规范的先前版本中被称为“模板字符串”。

略。

### 图片懒加载

```javascript
let imgList = [...document.querySelectorAll('img')]
let length = imgList.length

const imgLazyLoad = function() {
    let count = 0
    return (function() {
        let deleteIndexList = []
        imgList.forEach((img, index) => {
            let rect = img.getBoundingClientRect()
            if (rect.top < window.innerHeight) {
                img.src = img.dataset.src
                deleteIndexList.push(index)
                count++
                if (count === length) {
                    document.removeEventListener('scroll', imgLazyLoad)
                }
            }
        })
        imgList = imgList.filter((img, index) => !deleteIndexList.includes(index))
    })()
}

// 这里最好加上防抖处理
document.addEventListener('scroll', imgLazyLoad)
```

