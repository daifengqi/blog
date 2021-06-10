# JavaScript期约Promise专题

# JavaScript Topic of Promise

## 2021 年 5 月 23 日

## May 23 2021



### Promise入门

Promise是一种异步编程的解决方案。

Promise接受一个函数的输入，这个函数必须拥有两个参数，`(resolve, reject)`，这两个参数同时又是两个函数，如下，

```js
const pro = new Promise((resolve, reject) => {
  resolve();
})
```

传入resolve的参数可以在then后继续使用，如下，

```js
const pro = new Promise((resolve, reject) => {
  let foo = 1;
  resolve(foo);
}).then((bar) => {
  console.log(bar + 1); // 2
});
```

then中传入的参数可以对结果重命名，以满足接下来函数体的需要。

> 相反，如果调用reject，则会进入`.catch`块中，处理错误。

**🌟“同步”获取Promise数据**

Promise内产生的异步调用结果是无法在同步代码里使用的，这很好理解，因为异步调用是在代码同步执行后的<u>微任务</u>中进行，换句话说，**异步调用的代码在同步代码块中是还未执行的**，也即Promise还处于pending状态，如下，

```javascript
const value = Promise.resolve().then(() => {
  let value = 1;
  console.log("异步调用中的value值：", value);
});

console.log("同步调用中的value值：", value);
```

以上代码的打印结果为，

```
同步调用中的value值： Promise { <pending> }
异步调用中的value值： 1
```

那有些场景我们就希望在同步代码中使用异步结果呢，这个时候，**就必须把所有同步代码连同异步代码封装在一起，作为异步调用**，如下，

```js
const aysncCall = async () => {
  const value = await Promise.resolve().then(() => {
    let value = 1;
    console.log("异步调用中的value值：", value);
    return value;
  });

  console.log("同步调用中的value值：", value);
};

aysncCall();
```

以上代码充分利用了ES7的新特性及语法`async、await`，`async`函数体本身会异步调用，当其中碰到`await`时，又会去先执行**await后面的异步代码块**，并且将返回值赋值给`await`左边的对象，注意，这里不返回期约，而是后面异步调用的结果，**这就是await起到的效果**。

在整个异步的函数中，其定义的`value`值也就可以假装被“同步”地拿到，这样就起到了我们预期的效果。

### 手写Promise

先来看Promise的规范，

> [PromiseA+标准](https://promisesaplus.com/)
>
> [中文版](https://www.ituring.com.cn/article/66566)

首先，Promise有如下初始状态，

```javascript
function Promise(fn) {
  var self = this;
  self.status = 'pending';         // Promise初始状态为pending
  self.data = undefined;           // Promise的值
  self.onFulfilledCallback = [];   // Promise resolve回调函数集合
  self.onRejectedCallback = [];    // Promise reject回调函数集合
 
  try {
    fn(resolve, reject); // 执行传进来的函数，传入resolve, reject参数
  } catch (e) {
    reject(e);
  }
}
```

当你`new Promise()`的时候，传入的参数必须是`(resolve, reject) => {...}`这样格式的参数，分别对应“执行”和“拒绝”态。

```javascript
function resolve(value) {
  if (self.status === 'pending') {
    self.status = 'resolved';
    self.data = value;
    for (var i = 0; i < self.onFulfilledCallback.length; i++) {
      self.onFulfilledCallback[i](value);
    }
  }
}

function reject(reason) {
  if (self.status === 'pending') {
    self.status = 'rejected';
    self.data = reason;
    for (var i = 0; i < self.onRejectedCallback.length; i++) {
      self.onRejectedCallback[i](reason);
    }
  }
}
```

**🌟Promise.then**

Promise对象有一个`then`方法，用来注册Promise对象状态确定后的回调。这里需要将then方法定义在Promise的原型上，

```javascript
Promise.prototype.then = function(onFulfilled, onRejected) {
  var self = this;

  // 根据标准，如果then的参数不是function，则我们需要忽略它
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : function(v) { return v};
  onRejected = typeof onRejected === 'function' ? onRejected : function(r) { return r };
```

先实现`then`之前Promise的结果是`resolve`和`reject`的情况，

```javascript
if (self.status === 'resolved') {
  // 这里promise的状态已经确定是resolved，所以调用onResolved
  return new Promise(function(resolve, reject) {
    // 根据标准，需要异步执行
    setTimeout(function() {
      try {
        // ret是onFulfilled的返回值
        var ret = onFulfilled(self.data);
        if (ret instanceof Promise) {
          // 如果ret是一个promise，则取其值作为新的promise的结果
          ret.then(resolve, reject);
        } else {
          // 否则，以它的返回值作为新的promise的结果
          resolve(ret);
        }
      } catch (e) {
        // 如果出错，以捕获到的错误作为promise2的结果
        reject(e);
      }
    });
  });
}

// 这里的逻辑跟前面一样，不再赘述
if (self.status === 'rejected') {
  return new Promise(function(resolve, reject) {
    setTimeout(function() {
      try {
        var ret = onRejected(self.data);
        if (ret instanceof Promise) {
          ret.then(resolve, reject);
        } else {
          reject(ret);
        }
      } catch (e) {
        reject(e);
      }
    });
  });
}
```

实现`pending`的情况，

```javascript
if (self.status === 'pending') {
  return new Promise(function(resolve, reject) {
    self.onFulfilledCallback.push(function(value) {
      setTimeout(function() {
        try {
          var ret = onFulfilled(self.data);
          if (ret instanceof Promise) {
            ret.then(resolve, reject);
          } else {
            resolve(ret);
          }
        } catch (e) {
          reject(e);
        }
      });
    });

    self.onRejectedCallback.push(function(reason) {
      setTimeout(function() {
        try {
          var ret = onRejected(self.data);
          if (ret instanceof Promise) {
            ret.then(resolve, reject);
          } else {
            reject(ret);
          }
        } catch (e) {
          reject(e);
        }
      });
    });
  });
}
```

顺便实现一下catch方法 ，

```javascript
Promise.prototype.catch = function(onRejected) { 
  return this.then(null, onRejected); 
}
```

**🌟Promise.resolve**

> `Promise.resolve(value)`方法返回一个以给定值解析后的[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象。如果这个值是一个 promise ，那么将返回这个 promise ；如果这个值是thenable（即带有[`"then" `](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)方法），返回的promise会“跟随”这个thenable的对象，采用它的最终状态；否则返回的promise将以此值完成。此函数将类promise对象的多层嵌套展平。

```javascript
Promise.resolve(value) {
  if (value instanceof Promise) {
    return value;
  } else if (typeof value === 'object' 
             && typeof value.then === 'function') {
    return new Promise(resolve => {
      value.then(resolve);
    });
  } else {
    return new Promise(resolve => resolve(value));
  }
}
```

**🌟Promise.reject**

> `Promise.reject()`方法返回一个带有拒绝原因的`Promise`对象。

```javascript
Promise.reject = function(reason) {
    return new Promise((resolve, reject) => reject(reason))
}
```

**🌟Promise.all**

> Promise.all() 方法接收一个promise的iterable类型（注：Array，Map，Set都属于ES6的iterable类型）的输入，并且只返回一个`Promise`实例， 那个输入的所有promise的resolve回调的结果是一个数组。这个`Promise`的resolve回调执行是在所有输入的promise的resolve回调都结束，或者输入的iterable里没有promise了的时候。它的reject回调执行是，只要任何一个输入的promise的reject回调执行或者输入不合法的promise就会立即抛出错误，并且reject的是第一个抛出的错误信息。

解释一下，

- 传入的所有 Promsie 都是 fulfilled，则返回由他们的值组成的，状态为 fulfilled 的新 Promise；
- 只要有一个 Promise 是 rejected，则返回 rejected 状态的新 Promsie，且它的值是第一个 rejected 的 Promise 的值；
- 只要有一个 Promise 是 pending，则返回一个 pending 状态的新 Promise；
- 输入的参数arr只要是一个可迭代对象就可以，所以用`Array.from`转换一下；

```javascript
Promise.all = function(arr = Array.from(arr)) {
    let index = 0, result = [];
    return new Promise((resolve, reject) => {
        arr.forEach((p, i) => {
            Promise.resolve(p).then(val => {
                index++;
                result[i] = val;
                if (index === arr.length) {
                    resolve(result);
                }
            }, err => {
                reject(err);
            })
        })
    })
}
```

**Promise.race**

> `Promise.race(iterable)` 方法返回一个 promise，一旦迭代器中的某个promise解决或拒绝，返回的 promise就会解决或拒绝。

```javascript
Promise.race = function(arr = Array.from(arr)) {
    return new Promise((resolve, reject) => {
        arr.forEach(p => {
            Promise.resolve(p).then(val => {
                resolve(val)
            }, err => {
                rejecte(err)
            })
        })
    })
}
```

**🌟Promise.any**

`Promise.any()` 是 ES2021 新增的特性，它接收一个 `Promise` 可迭代对象（例如数组），

- 只要其中的一个 `promise` 成功，就返回那个已经成功的 `promise`
- 如果可迭代对象中没有一个 `promise` 成功（即所有的 `promises` 都失败/拒绝），就返回一个失败的 `promise` 和 `AggregateError` 类型的实例，它是 `Error` 的一个子类，用于把单一的错误集合在一起

`Promise.any()`的应用场景：从世界各地最快的服务器检索资源。

```javascript
  function getUser(endpoint) {
    return fetch(`https://superfire.${endpoint}.com/users`)
      .then(response => response.json());
  }
  
  const promises = [getUser("jp"), getUser("uk"), getUser("us"), getUser("au"), getUser("in")]
  
  Promise.any(promises).then(value => {
    console.log(value)
  }).catch(err => {
    console.log(err);
  })
```

实现如下，

```javascript
Promise.any = function(promises = Array.from(promises)){
  return new Promise((resolve,reject)=>{
    promises = Array.isArray(promises) ? promises : []
    let len = promises.length
    // 用于收集所有 reject 
    let errs = []
    // 如果传入的是一个空数组，那么就直接返回 AggregateError
    if(len === 0) return reject(new AggregateError('All promises were rejected'))
    promises.forEach((promise)=>{
      promise.then(value=>{
        resolve(value)
      },err=>{
        len--
        errs.push(err)
        if(len === 0){
          reject(new AggregateError(errs))
        }
      })
    })
  })
}
```

