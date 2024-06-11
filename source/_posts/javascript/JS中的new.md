---
title: 一起搞清楚JS中的new到底做了什么，并模拟实现一个new
urlname: js-new
tags:
  - JavaScript
categories:
  - [前端, JavaScript]
  - [前端, 原理源码]
abbrlink: 18372
date: 2021-04-13 23:00:06
---

> 作者：Hanpeng_Chen
>
> 公众号：**前端极客技术**

new关键字对于前端开发者来说是比较常见的操作，在互联网大厂的面试中，有时候会要求手写实现new。接下来我们一起看看new到底做了什么？如何模拟实现？

## new原理介绍

### new概念
关于new关键字，MDN上是这样描述的：

> `new`关键字创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。

上面这句话也许有些难懂，我们先来看下面这段代码：

```js
let Parent = function(name, age) {
  this.name = name;
  this.age = age;
};
Parent.prototype.getName = function(){
  return this.name;
}
let child = new Parent('张三', 25);
console.log(child.getName());
console.log(child.age);
```

从这个例子中，我们可以看到：实例child可以访问到Parent构造函数里的属性，也可以访问到Parent.prototype中的属性。

从输出结果可以看出：child是一个通过Parent这个构造函数生成的一个实例对象。

> 那么关于new关键字的作用，我们可以理解为就是执行一个构造函数，返回一个实例对象，在new的过程中，根据构造函数的情况，来确定是否可以接受参数传递。


### 不使用new关键字会发生什么
我们对上面代码进行改造一下，去掉new，会有什么变化？

```js
let Parent = function(name, age) {
  this.name = name;
  this.age = age;
}
let child1 = Parent('张三', 25)
console.log(child1) // undefined
console.log(age) // 25
console.log(child1.name) // Cannot read property 'name' of undefined
```

从上面代码我们可以看到，没有使用new关键词，返回的结果是`undefined`。我们直接打印age可以获取到值，是因为JavaScript代码在默认情况下this的指向是window。


### 构造函数返回一个对象会发生什么？
如果构造函数中有return一个对象的操作，结果会发生什么变化？我们再来看下面这段改造后的代码：

```js
function Parent(name) {
  this.name = name;
  return {age: 30}
}
let p = new Parent('张三')
console.log(p); // { age: 30 }
console.log(p.name); // undefined
console.log(p.age); // 30
```

从上述代码的执行结果看到：如果构造函数return出来的是一个和this无关的对象时，new命令会直接返回这个新对象，而不是生成一个绑定了最新this的新对象并返回出来。

如果构造函数return回来的不是一个对象，那么还是会按照new的实现步骤，返回新生成的对象。我们通过下面的代码验证一下：
```js
function Parent(name) {
  this.name = name;
  return 'Chinese'
}
let p = new Parent('张三')
console.log(p); // Parent { name: '张三' }
console.log(p.name); // 张三
```

### 小结：new到底做了什么？
new在这个生成实例过程中到底进行了哪些步骤来实现？我将其总结为下面几个步骤：

- 创建一个空的简单JavaScript对象，即 `{}`；
- 将构造函数的作用域赋给新对象（this指向新对象）；
- 执行构造函数中的代码（为新对象添加属性）；
- 如果该函数没有返回对象，则返回this。



## new的实现
结合上面的内容，我们一起来手写实现new。

在开始写代码之前，我们先确定new被调用后大致做了那几件事：

- 让实例可以访问到私有属性；
- 让实例可以访问构造函数原型（constructor.prototype）所在原型链上的属性；
- 构造函数返回的最后结果是引用数据类型。

知道了new被调用后做了哪些事情，那么我们就可以开始手写实现它了。实现代码如下：

```js
function _new(ctor, ...args) {
  if (typeof ctor !== 'function') {
    throw 'ctor must be a function';
  }
  // 创建新的对象
  let newObj = new Object();
  // 让新创建的对象可以访问构造函数原型（constructor.prototype）所在原型链上的属性；
  newObj.__proto__ = Object.create(ctor.prototype);
  // 将构造函数的作用域赋给新对象（this指向新对象）；
  // 执行构造函数中的代码
  let res = ctor.apply(newObj, [...args]);

  let isObject = typeof res === 'object' && res !== null;
  let isFunction = typeof res === 'function';
  return isObject || isFunction ? res : newObj;
}
```

对我们上面模拟实现的new方法进行测试，代码如下：
```js
let Parent = function(name, age) {
  this.name = name;
  this.age = age;
};
Parent.prototype.getName = function(){
  return this.name;
}
let p = _new(Parent, 'zhangsan', 25);
console.log(p); // Parent { name: 'zhangsan', age: 25 }
console.log(p.age); // 25 
console.log(p.getName()); // zhangsan


function Parent1(name) {
  this.name = name;
  return {age: 30}
}
let p1 = _new(Parent1, 'lisi')
console.log(p1); // { age: 30 }
console.log(p1.name); // undefined
console.log(p1.age); // 30


function Parent2(name) {
  this.name = name;
  return 'Chinese'
}
let p2 = _new(Parent2, 'wangwu')
console.log(p2); // Parent { name: 'wangwu' }
console.log(p2.name); // wangwu
```

从测试结果可以看出，我们模拟实现的_new方法和原生的new关键字调用结果是一样的。

到这里手写模拟实现new就结束了，你学会了吗？

如有发现不妥或可改进之处，欢迎留言指出。


> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术**，我们一起学习一起进步。


![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/%E5%89%8D%E7%AB%AF%E6%9E%81%E5%AE%A2%E6%8A%80%E6%9C%AF%E4%BA%8C%E7%BB%B4%E7%A0%81.png)