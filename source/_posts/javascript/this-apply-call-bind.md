---
title: JS中的apply、call、bind你掌握了吗？一起来手写实现这三个方法吧
urlname: handwriting-apply-call-bind
tags:
  - JavaScript
  - 手写源码
categories:
  - [前端, JavaScript]
  - [前端, 原理源码]
abbrlink: 46930
date: 2021-04-21 07:35:39
---

> 作者：Hanpeng_Chen
>
> 公众号：**前端极客技术**


apply、call和bind这三个方法在函数原型链中是比较重要的概念，和this关键字密切相关。如果你对这三个方法还不是很清楚的话，那么认真地阅读这篇文章吧，让我们一起来彻底掌握它们吧！

在开始介绍apply/call/bind 这三个方法之前，我们需要先来了解一下JS中的this指向问题。

## this指向
在ES5中，我们只要牢记一句话：this永远指向最后调用它的那个对象，那么对于this的指向问题就比较好理解了。

### 直接函数调用

我们先来看一个例子：
```js
var name = 'global name';
function getName() {
  var name = 'function name';
  console.log('in function', this);
  console.log(this.name);
}
getName();
console.log('out function', this)
```
在浏览器控制台执行结果如下图所示：
![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/20210420214029.png)

我们可以看到在全局环境中直接调用函数getName()相当于`window.getName()`，调用它的是全局对象window，所以this对象指向全局对象，也就是window对象。

### 对象函数调用

我们再看下面这个例子：
```js
var name = 'global name';
var obj = {
  name: 'object name',
  getName: function() {
    console.log(this); // {name: "object name", getName: ƒ}
    console.log(this.name); // object name
  }
}
obj.getName();
```

在上面这个例子中，函数 getName 时对象 obj 调用的，所以在 getName 函数中打印的 this 是 obj 对象本身，this.name 是 `object name`。

我们将上面的代码进行如下改造：
```js
var name = "global name";
function getName() {
  console.log(this.name);
}
var obj = {
  name: "object name"
};
obj.getName = getName;
obj.getName(); // object name
getName(); // global name
```

在上面代码中，我们将 `getName` 函数的指针赋值给 `obj.getName`，`obj.getName()` 是对象函数调用方法，this指向obj本身，所以打印的this.name是 object name。

而全局环境调用`getName()`，等同于 `window.getName()`，打印的也就是 global name。

从上面两个示例我们可以总结出：对象函数调用，this指针指向调用函数的对象本身。

### 构造函数调用

接下来我们来看下构造函数里面this指针的指向问题，还是先看下面的示例代码：
```js
function NewObject(name) {
  this.name = name;
  console.log(this.name)
}
var obj = new NewObject('obj')
```

从上面例子我们可以看出：构造函数中的this指向新创建的对象本身，所以上面例子打印出来的name为 obj。


### 小结
- 直接函数调用，this指针指向全局环境；
- 对象函数调用，this指针指向调用函数的对象本身；
- 构造函数调用，this指针指向新创建的对象本身。


## 改变this的指向的几种方法

改变this的指向的方法主要有以下几种：
- 使用ES6中的箭头函数
- 在函数内部使用 _this = this
- 使用apply、call、bind
- new实例化一个对象

### 箭头函数
箭头函数中的this只取决于它外面的第一个不是箭头函数的函数的this，并且箭头函数的this一旦绑定了上下文，就不会被任何代码改变。
```js
function foo() {
    return () => {
        return () => {
          	console.log(this.a);  
        };  
    };
}
var a = 2;
console.log(foo()()()); // 2
```

### 在函数内部使用 _this = this
先将调用这个函数的对象保存在变量 _this 中，然后在函数中都使用这个 _this，这样 _this 就不会改变了。

```js
var name = 'global name'
var obj = {
  name: 'obj name',
  fn1: function () {
    console.log(this.name);
  },
  fn2: function () {
    var _this = this
    setTimeout(function() {
      _this.fn1()
    }, 1000)
  }
}
obj.fn2() // obj name
```

在上面的例子中，如果fn2函数不通过_this变量保存上下文，而是在setTimeout延时函数中直接使用this的话，这时的this指向的是window，但是window对象上并没有定义fn1函数，程序就会报错。

### new实例化一个对象
这个我们在上面构造函数调用中已经介绍过了，new实例化一个对象，this指针指向新创建的对象本身。


接下来我们一起来看看apply、call、bind的使用及其原理。

## apply、call、bind 基本介绍

**基本语法：**
```js
fn.apply(thisArg, [p1, p2, ...])
fn.call(thisArg, p1, p2, ...)
fn.bind(thisArg, p1, p2, ...)
```

这三个方法是挂在Function对象上的，**调用这三个方法的必须是一个函数。**

这三个方法的共有作用：**改变函数fn的this指向**。

call和apply的区别在于:传参的写法不同，apply的第2个参数是数组，call则是从第2个至第n个都是给fn的传参。

bind和上面两个方法又有所不同，bind虽然改变了fn的this指向，但不是马上执行，而apply和call是在改变了this指向后立即执行。


## apply/call/bind的核心理念：借用方法

我们先来看下面apply、call、bind这三个方法的简单示例：
```js
let dog = {
  name: 'dog',
  getDetail: function (count) {
    return `${this.name} has ${count} legs`
  }
}

let bird = {
  name: 'bird'
}
let frog = {
  name: 'frog'
}

console.log(dog.getDetail(4)); // dog has 4 legs
console.log(dog.getDetail.apply(bird, [2])); // bird has 2 legs
console.log(dog.getDetail.call(frog, 3)); // frog has 3 legs
let bird1 = dog.getDetail.bind(bird);
console.log(bird1(2)); // bird has 2 legs
```

在上面代码中，dog对象有一个 `getDetail` 的方法，对象bird和frog也需要临时使用同样的方法，那么这时候我们可以借用dog对象的getDetail方法，这样既能达到目的，也能节省重复定义，节约内存空间。

这就是这三个方法的核心理念：借用方法。

## 手写实现apply、call和bind
在大厂面试中，手写实现一个apply/call/bind是一道高频面试题，接下来我们一起来实现这三个方法：

### apply的实现
根据上面apply的原理和其借用方法的理念，我们来整理一下实现 apply 方法的思路：

1. 通过设置context的属性，将函数的this指向隐式绑定到 context上；

2. 通过隐式绑定执行函数并传递参数；

3. 删除临时属性，返回函数执行结果。

具体实现代码如下：

```js
Function.prototype.apply = function(context, args) {
  if (context === null || context === undefined) {
    context = window
  } else {
    context = Object(context) // 值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的实例对象
  }
  context.fn = this;
  let result = eval('context.fn(...args)');
  // 删除添加的函数
  delete context.fn;
  return result;
}
```

从上面的代码可以看出，实现apply的关键在于 `eval` 这行代码。其中显示了用context这个临时变量来指定上下文，然后还是通过执行 `eval` 来执行 `context.fn` 这个函数，最后返回result。


如果不使用eval，我们还有下面的实现方式：
```js
Function.prototype.apply = function (context, args) {
  if (context === null || context === undefined) {
    context = window;
  } else {
    context = Object(context); // 值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的实例对象
  }
  // 给context新增一个独一无二的属性以免覆盖原来属性
  let fn = Symbol();
  context[fn] = this;
  // 通过隐式绑定的方式调用函数
  let result = context[fn](...args);
  // 删除添加的函数
  delete context[fn];
  return result;
};
```

使用Symbol临时存储函数，避免了跟上下文对象的原属性冲突的风险。

### call的实现

call和apply基本原理差不多，只是参数存在区别，这里我们不再细说，直接看call方法的实现代码：

```js
Function.prototype.call = function(context, ...args) {
  if (context === null || context === undefined) {
    context = window
  } else {
    context = Object(context)
  }
  context.fn = this;
  let result = eval('context.fn(...args)');
  delete context.fn;
  return result;
}
```


### bind的实现
bind方法和其它两个方法的区别在于：bind方法是返回一个函数，而其它两个方法是直接返回执行结果。

实现思路和call基本相同，因为bind不需要返回执行结果，所以不需要用eval，而是需要通过返回一个函数的方式将结果返回，之后再执行这个结果，得到想要的结果。

具体实现思路如下：

1. 拷贝源函数：
  - 通过变量存储源函数
  - 使用 `Object.create` 复制源函数的prototype给fbound

2. 返回拷贝的函数

3. 调用拷贝的函数
  - new调用判断：通过 `instanceof` 判断函数是否通过 new 调用，来决定绑定的 context
  - 绑定 this，传递参数
  - 返回源函数的执行结果

```js
Function.prototype.bind = function (context, ...args) {
  if (typeof this !== 'function') {
    throw new Error('this must be a function')
  }
  var self = this;
  var fbound = function() {
    return self.call(this instanceof fbound ? this : Object(context), ...args, ...arguments);
    // return self.call(this instanceof fbound ? this : Object(context), args.concat(Array.prototype.slice.call(arguments)))
  }
  if (this.prototype) {
    fbound.prototype = Object.create(this.prototype);
  }
  return fbound;
}
```


从上面的代码中可以看到，实现 bind 的核心在于返回的时候需要返回一个函数，故这里的 fbound 需要返回，但是在返回的过程中原型链对象上的属性不能丢失。因此这里需要用Object.create 方法，将 this.prototype 上面的属性挂到 fbound 的原型上面，最后再返回 fbound。这样调用 bind 方法接收到函数的对象，再通过执行接收的函数，即可得到想要的结果。




看到这里，apply、call、bind这三个方法的实现你是否已经完全掌握了？如果没有的话那就自己动手多写几次吧！


> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术**，我们一起学习一起进步。