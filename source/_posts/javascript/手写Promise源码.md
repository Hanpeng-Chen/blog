---
title: 一起手写实现符合Promise/A+规范的Promise源码
urlname: handwriting-promise-accord-with-promise-a-plus-specification
tags:
  - JavaScript
  - 原理源码
categories:
  - [前端, JavaScript]
  - [前端, 原理源码]
abbrlink: 46892
date: 2021-03-28 10:09:59
---

> 作者：Hanpeng_Chen
>
> 公众号：**前端极客技术**

Promise是JavaScript中异步编程的核心内容，也是前端面试的高频问题。关于Promise的基本用法这里不再详细介绍，接下来我们一起来实现一个符合 Promise/A+ 规范的Promise。

在开始写代码之前，我们要知道Promise/A+规范都有哪些内容。

## Promise/A+ 规范

https://promisesaplus.com/

上面是Promise/A+规范的官方地址，是一个英文版本，大家可以自行点击阅读。下面我把规范中比较关键的点进行说明：

### 术语

> 1. promise 是一个具有then方法的对象或函数，它的行为符合该规范。
>
> 2. thenable 是一个定义了then方法的对象或函数。
>
> 3. value 可以是任何一个合法的JavaScript的值，包括undefined、thenable、promise。
>
> 4. exception 是一个异常，是在Promise里面可以用throw语句抛出的值。
>
> 5. reason 是一个Promise里reject之后返回的拒绝原因。

### 状态

- 1. 一个Promise有三种状态：pending、fulfilled、rejected。
- 2. 当状态为pending时，可以转换为 fulfilled 或者 rejected。
- 3. 当状态为 fulfilled 时，就不能再变为其他状态，必须返回一个不能再改变的值。
- 4. 当状态为 rejected 时，就不能再变为其他状态，必须有一个promise被reject的原因，原因值也不能改变。

### then方法
Promise必须拥有一个then方法，来访问最终的结果。

then方法有两个参数：
```js
promise.then(onFulfilled, onRejected)
```

**onFulfilled 和 onRejected**
onFulfilled和onRejected两个都是可选参数，两者都必须是函数类型。

如果onFulfilled是函数，必须在promise变成fulfilled时，调用 onFulfilled，参数是promise的value，onFulfilled只能被调用一次。

如果onRejected是函数，必须在promise变成rejected时，调用 onRejected，参数是promise的reason，onRejected只能被调用一次。

**then方法可以被多次调用**
then方法可以被一个Promise调用多次，**且必须返回一个Promise对象**。

如果promise变成了 fulfilled态，所有的onFulfilled回调都需要按照then的顺序执行

如果promise变成了 rejected态，所有的onRejected回调都需要按照then的顺序执行


Promise/A+规范的主要部分就是上面这些，接下来我们一起动手实现一个Promise。



## 实现 Promise

### 构造函数
Promise构造函数接受一个 executor 函数，executor函数执行完同步或异步操作后，调用它的两个参数resolve和reject。

```js
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";


function Promise(executor) {
  let self = this;
  self.status = PENDING;
  self.data = undefined; // Promise的值
  self.onResolvedCallback = []; // Promise resolve时的回调函数
  self.onRejectedCallback = []; // Promise rejected的回调函数集

  function resolve(value) {

  }
  function reject(reason) {

  }

  try {
    executor(resolve, reject)
  } catch(e) {
    reject(e)
  }
}
```

Promise构造函数中定义了resolve和reject两个函数，下面一起来看看这两个函数如何实现？

从规范中我们知道：resolve和reject函数主要是返回对应状态的值value或者reason，并将Promise内部的状态从pending变成对应的状态，并且在这个状态改变之后是不可逆的。

那么这两个函数要如何实现，可以看下面这段代码：

```js
function Promise(executor) {
  ...

  function resolve(value) {
    if (self.status === PENDING) {
      self.status = FULFILLED;
      self.data = value;
      for(let i = 0; i < self.onResolvedCallback.length; i++) {
        self.onResolvedCallback[i](value)
      }
    }
  }
  function reject(reason) {
    if (self.status === PENDING) {
      self.status = REJECTED;
      self.data = reason;
      for(let i = 0; i < self.onRejectedCallback.length; i++) {
        self.onRejectedCallback[i](reason);
      }
    }
  }
  ...
}
```

### then方法的实现

then方法接收两个参数，分别是Promise成功的回调 onFulfilled 和 失败的回调 onRejected。

如果调用then时，promise已经成功，则执行onFulfilled，并将value值作为参数传递进去；

如果promise已经失败，执行onRejected，将失败的原因reason作为参数传递进去。

如果promise的状态是pending，需要将onFulfilled和onRejected函数存放起来，等待状态确定后，再依次将对应的函数执行(发布订阅)

promise 可以then多次，promise 的then 方法返回一个 promise

我们把then方法实现的思路过了一遍，下面来看其实现代码：
```js
Promise.prototype.then = function(onFulfilled, onRejected) {
  let self = this;
  let promise2;

  // 根据规范，如果then的参数不是function，则需要忽略它
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
  onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason };

  if (self.status === FULFILLED) {
    // 如果promise状态确定为fulfilled，调用onFulFilled，但代码执行中可能会抛出，所以将其包裹在try/catch代码块中
    return promise2 = new Promise(function(resolve, reject) {
      try {
        let x = onFulfilled(self.data)
        if (x instanceof Promise) {
          x.then(resolve, reject)
        }
        resolve(x)
      } catch(e) {
        reject(e)
      }
    })
  }
  if (self.status === REJECTED) {
    return promise2 = new Promise(function(resolve, reject) {
      try {
        let x = onRejected(self.data);
        if (x instanceof Promise) {
          x.then(resolve, reject)
        }
      } catch(e) {
        reject(e)
      }
    })
  }
  if (self.status === PENDING) {
    return promise2 = new Promise(function(resolve, reject) {
      self.onResolvedCallback.push(function(value) {
        try {
          let x = onFulfilled(self.data);
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch(e) {
          reject(e)
        }
      })
      self.onRejectedCallback.push(function(reason) {
        try {
          let x = onRejected(self.data)
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch(e) {
          reject(e)
        }
      })
    })
  }
}
```

到这里一个符合Promise/A+规范的then方法基本实现了。

### 优化
在Promise/A+规范中，onFulfilled和onRejected两个函数需要异步调用。

并且规范中提到，要支持不同的Promise进行交互，关于不同的Promise交互详细说明点击下方链接查看：

https://promisesaplus.com/#point-46

基于上面两点，我们对上面实现的then方法中执行Promise的方法进行优化，在处理Promise进行resolve或reject时，加上 setTimeout(fn, 0)

优化后的完整代码如下：
```js
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

function Promise(executor) {
  let self = this;
  self.status = PENDING;
  self.data = undefined; // Promise的值
  self.onResolvedCallback = []; // Promise resolve时的回调函数
  self.onRejectedCallback = []; // Promise rejected的回调函数集

  function resolve(value) {
    if (value instanceof Promise) {
      return value.then(resolve, reject);
    }
    setTimeout(function () {
      if (self.status === PENDING) {
        self.status = FULFILLED;
        self.data = value;
        for (let i = 0; i < self.onResolvedCallback.length; i++) {
          self.onResolvedCallback[i](value);
        }
      }
    }, 0);
  }
  function reject(reason) {
    setTimeout(function () {
      if (self.status === PENDING) {
        self.status = REJECTED;
        self.data = reason;
        for (let i = 0; i < self.onRejectedCallback.length; i++) {
          self.onRejectedCallback[i](reason);
        }
      }
    }, 0);
  }

  try {
    executor(resolve, reject);
  } catch (e) {
    reject(e);
  }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
  let self = this;
  let promise2;

  // 根据规范，如果then的参数不是function，则需要忽略它
  onFulfilled =
    typeof onFulfilled === "function" ? onFulfilled : function(v) {
      return v;
    };
  onRejected =
    typeof onRejected === "function"
      ? onRejected
      : function(r) {
        throw r;
      };

  if (self.status === FULFILLED) {
    // 如果promise状态确定为fulfilled，调用onFulFilled，但代码执行中可能会抛出，所以将其包裹在try/catch代码块中
    return promise2 = new Promise(function (resolve, reject) {
      setTimeout(function(){
        try {
          let x = onFulfilled(self.data);
          resolvePromise(promise2, x, resolve, reject)
        } catch (e) {
          reject(e);
        }
      })
    });
  }
  if (self.status === REJECTED) {
    return promise2 = new Promise(function (resolve, reject) {
      setTimeout(function(){
        try {
          let x = onRejected(self.data);
          resolvePromise(promise2, x, resolve, reject)
        } catch (e) {
          reject(e);
        }
      })
    });
  }
  if (self.status === PENDING) {
    return promise2 = new Promise(function (resolve, reject) {
      self.onResolvedCallback.push(function (value) {
        try {
          let x = onFulfilled(self.data);
          resolvePromise(promise2, x, resolve, reject)
        } catch (e) {
          reject(e);
        }
      });
      self.onRejectedCallback.push(function (reason) {
        try {
          let x = onRejected(self.data);
          resolvePromise(promise2, x, resolve, reject)
        } catch (e) {
          reject(e);
        }
      });
    });
  }
};


function resolvePromise(promise2, x, resolve, reject) {
  let then;
  let thenCalledOrThrow = false;
  if (promise2 === x) {
    return reject(new TypeError('Chaining cycle detected for promise'))
  }
  if (x instanceof Promise) {
    if (x.status === PENDING) {
      x.then(function(v) {
        resolvePromise(promise2, v, resolve, reject)
      }, reject)
    } else {
      x.then(resolve, reject)
    }
    return
  }
  if ((x !== null) && (typeof x === 'object' || typeof x === 'function')) {
    try {
      then = x.then;
      if (typeof then === 'function') {
        then.call(x, function(y) {
          if (thenCalledOrThrow) return; // 已经调用过
          thenCalledOrThrow = true;
          return resolvePromise(promise2, y, resolve, reject)
        }, function(r) {
          if (thenCalledOrThrow) return;
          thenCalledOrThrow = true;
          return reject(r);
        })
      } else {
        resolve(x)
      }
    } catch(e) {
      if (thenCalledOrThrow) return;
      thenCalledOrThrow = true;
      return reject(e);
    }
  } else {
    resolve(x)
  }
}

Promise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected)
}

module.exports = Promise;
```

## 测试

有专门的测试脚本可以测试我们写的Promise是否符合Promise/A+规范。

首先在Promise实现的代码中，暴露一个deferred方法：
```js
Promise.defer = Promise.deferred = function(){
  let dfd = {};
  dfd.promise = new Promise(function(resolve, reject) {
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd;
}
```

在Promise源码的目录执行npm安装和执行测试命令：
```js
npm i -g promises-aplus-tests

promises-aplus-tests Promise.js
```

![](https://image.chenhanpeng.com/static/blog-images/blogImages/2021/20210323235204.png)

promises-aplus-tests中共有872条测试用例。从上面的执行结果可以看出，我们手写的Promise通过所有用例。


## Promise其他方法的实现
原生的Promise还提供了一些其他方法，比如：
- Promise.resolve()
- Promise.reject()
- Promise.prototype.catch()
- Promise.prototype.finally()
- Promise.all()
- Promise.race()
- ...


在面试中也经常碰到面试官要求手写实现这些方法，接下来我们一起看看这几个方法如何实现：

### Promise.resolve

Promise.resolve(value)方法返回一个给定值解析后的Promise对象。

> - 如果 value 是个 thenable 对象，返回的promise会“跟随”这个thenable的对象，采用它的最终状态
>
> - 如果传入的value本身就是promise对象，那么Promise.resolve将不做任何修改、原封不动地返回这个promise对象。
>
> - 其他情况，直接返回以该值为成功状态的promise对象。

```js
Promise.resolve = function (value) {
  if (value instanceof Promise) {
    return value
  }
  return new Promise((resolve, reject) => {
    if (value && typeof value === 'object' && typeof value.then === 'function') {
      setTimeout(() => {
        value.then(resolve, reject)
      })
    } else {
      resolve(value)
    }
  })
}
```

### Promise.reject
Promise.reject方法和Promise.resolve不同，Promise.reject()方法的参数，会原封不动地作为reject的理由，变成后续方法的参数。

```js
Promise.reject = function(reason) {
  return new Promise((resolve, reject) => {
    reject(reason)
  })
}
```

### Promise.prototype.catch
Promise.prototype.catch 用于指定出错时的回调，是特殊的then方法，catch之后，可以继续.then
```js
Promise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected)
}
```

### Promise.prototype.finally
不管成功还是失败，都会走到finally中,并且finally之后，还可以继续then。并且会将值原封不动的传递给后面的then.
```js
Promise.prototype.finally = function(cb) {
  return this.then((value) => {
    return Promise.resolve(cb()).then(() => {
      return value
    })
  }, (err) => {
    return Promise.resolve(cb()).then(() => {
      throw err;
    })
  })
}
```

### Promise.all
Promise.all(promises) 返回一个promise对象

如果传入的参数是一个空的可迭代对象，那么此promise对象回调完成(resolve),只有此情况，是同步执行的，其它都是异步返回的。

如果传入的参数不包含任何 promise，则返回一个异步完成.

promises 中所有的promise都promise都“完成”时或参数中不包含 promise 时回调完成。

如果参数中有一个promise失败，那么Promise.all返回的promise对象失败

在任何情况下，Promise.all 返回的 promise 的完成状态的结果都是一个数组

```js
Promise.all = function(promises) {
  promises = Array.from(promises); // 将可迭代对象转换成数组
  return new Promise((resolve, reject) => {
    let index = 0;
    let result = [];
    if (promises.length === 0) {
      resolve(result);
    } else {
      function processValue(i, value) {
        result[i] = value;
        if (++index === promises.length) {
          resolve(result);
        }
      }
      for (let i = 0; i < promises.length; i++) {
        Promise.resolve(promises[i]).then((data) => {
          processValue(i, data);
        }, (err) => {
          reject(err);
          return;
        })
      }
    }
  })
}
```

### Promise.race
Promise.race函数返回一个 Promise，它将与第一个传递的 promise 相同的完成方式被完成。它可以是完成（ resolves），也可以是失败（rejects），这要取决于第一个完成的方式是两个中的哪个。

如果传的参数数组是空，则返回的 promise 将永远等待。

如果迭代包含一个或多个非承诺值和/或已解决/拒绝的承诺，则 Promise.race 将解析为迭代中找到的第一个值。

```js
Promise.race = function(promises) {
  promises = Array.from(promises);
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      return;
    } else {
      for (let i = 0; i < promises.length; i++) {
        Promise.resolve(promises[i]).then((data) => {
          resolve(data);
          return;
        }, (err) => {
          reject(err);
          return;
        })
      }
    }
  })
}
```

本文所有源代码：[handwriting-promise](https://github.com/Hanpeng-Chen/html-js-demo-code/tree/main/handwriting-promise)

## 总结

第一次写难免遇到各种问题，只要对照规范多写几次，自己总结一下，你也可以很快写出通过测试的Promise源码。

如果你能根据PromiseA+的规范，写出符合规范的源码，那么我想，对于面试中的Promise相关的问题，都能够给出比较完美的答案。


> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术**，我们一起学习一起进步。