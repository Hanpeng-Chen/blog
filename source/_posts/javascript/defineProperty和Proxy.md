---
title: 学习Vue源码前需要了解的defineProperty和Proxy
urlname: javascript-defineProperty-and-proxy
tags:
  - JavaScript
categories:
  - - 前端
    - JavaScript
abbrlink: 24170
date: 2021-08-24 12:12:35
---

> 作者：Hanpeng_Chen
>
> 公众号：前端极客技术

## 前言

大家有使用Vue开发想必对响应式都有了解，知道Vue2是用`Object.defineProperty`实现数据劫持，进而实现的双向绑定。在已经发布快一年的Vue3中，数据响应式的实现由`Object.defineProperty` API改成了`Proxy` API。

接下来我们一起来看看这两个API的基本用法。

## defineProperty

`Object.defineProperty()` 是ES5提供的方法，该方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

### 语法

> Object.defineProperty(obj, prop, description)

### 参数

- obj：要在其上定义属性的对象
- prop：要定义或修改的属性的名称
- description：将被定义或修改的属性的描述符。是一个对象，具体为：
  - configurable：是否可以修改默认属性，类型为 boolean
  - enumerable：是否可以被枚举，类型为 boolean
  - writable：是否可以修改这个属性的值，类型为 boolean
  - value：初始值，可以是任意类型的值
  - get：被修饰的属性，在被访问的时候执行，类型为 Function
  - set：被修饰的属性，在被修改的时候执行，类型为 Function

description参数表示的属性描述符有两种主要形式：数据描述符和存取描述符。一个描述符只能是这两者中的一个，不能同时是两者。

数据描述符是一个具有值的属性，该值可以是可写的，也可以是不可写的。

存取描述符是由`get`和`set`函数所描述的属性。

两种描述符都是对象。

#### 两者均具有以下两种可选键值

**configurable**

当且仅当该属性的`configurable`键值为true时，该属性的描述符才能够被改变，同时该属性也能从对应的对象上被删除。默认值为false。

**enumerable**

当且仅当该属性的`enumerable`键值为true时，该属性才会出现再对象的枚举属性中。默认为false。

#### 数据描述符具有以下两种可选键值

**value**

该属性对应的值。可以是任何有效的JavaScript值。默认为 `undefined`。

**writable**

当且仅当该属性的 `writable` 键值为 true 时，属性的值，也就是上面的 value ，才能被赋值运算符改变。默认为false。

#### 存取描述符还具有以下可选键值

**get**

一个给属性提供 getter 函数的方法，如果没有getter为undefined。该方法返回值被用作属性值。默认为 undefined。

**set**

一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。该方法将接受唯一参数，并将该参数的新值分配给该属性。默认为 undefined。



### 示例
我们通过下面一个例子来看看defineProperty是如何在访问以及赋值的时候执行get和set。

```js
let obj = {}, value = null;
Object.defineProperty(obj, 'title', {
  get: function() {
    console.log('执行了 get 操作');
    return value;
  },
  set: function(newValue) {
    console.log('执行了 set 操作');
    value = newValue;
  }
})

obj.title = 'defineProperty demo'; // 执行了 set 操作
console.log(obj.title); // 执行了 get 操作 // defineProperty demo
obj.desc = 'xxxxxx'; // 不执行set  因为没有对desc这个属性进行劫持
console.log(obj.desc); // xxxxxx   不执行get，因为没有对desc这个属性进行劫持
```

> 从上面的例子我们可以看到，defineProperty  只能代理某个属性，如果没有对属性进行劫持代理，对其进行操作是不会走 get 和 set 方法

我们再来看看属性描述符是数据描述符的例子：

```js
Object.defineProperty({}, 'title', {
  value: 'demo',
  writable: true,
  enumerable: true,
  configurable: true
});
```

如果属性描述符同时是数据描述符和存取描述符两种形式，则会报错：
```js
// 报错：TypeError: Invalid property descriptor. Cannot both specify accessors and a value or writable attribute, #<Object>
Object.defineProperty({}, 'title', {
  value: 'demo',
  get: function() {
    return 'defineProperty demo'
  }
})
```

## Proxy

使用defineProperty只能重定义属性的get和set行为，到了ES6，提供了Proxy，可以重定义更多的行为，比如in、delete、函数调用等更多行为。

Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

### 语法

> var proxy = new Proxy(target, handler)

### 参数

- target：要使用 `Proxy` 包装的目标对象（可以是任何类型的对象，包括原生数组、函数，甚至另一个代理）。

- handler：一个通常以函数作为属性的对象，各属性中的函数分别定义了再执行各种操作时代理`p`的行为。回调方法的合集。
  -  handler.getPrototypeOf()
  -  handler.setPrototypeOf()
  -  handler.isExtensible()
  -  handler.preventExtensions()
  -  handler.getOwnPropertyDescriptor()
  -  handler.defineProperty()
  -  handler.has()
  -  handler.get(target, property)
  -  handler.set(target, property, value)
  -  handler.deleteProperty()
  -  handler.ownKeys()
  -  handler.apply()
  -  handler.construct()

从上面我们可以发现，Proxy也有get和set方法。和defineProperty相比，Proxy接收的target可以为任何类型的对象，包括原生数组、函数，甚至是另一个代理对象。

### 示例
先来看下get和set的例子：

```js
let proxy = new Proxy({}, {
  get: (obj, propKey) => {
    console.log(' get 操作')
    return obj[propKey]
  },
  set: (obj, propKey, value) => {
    console.log('设置 set 操作')
    obj[propKey] = value
  }
})

proxy.title = 'proxy demo' // 设置 set 操作
console.log(proxy.title) //  get 操作 // proxy demo
proxy.desc = 'xxxx' // 设置 set 操作
console.log(proxy.desc) // get 操作 // xxxx
```

如果对象时一个多层嵌套对象，我们来看下Proxy对其属性的监听：
```js
let obj = {
  a: 1,
  b: {
    a1: 11,
    b1: {
      a2: 21
    }
  }
}
let proxy = new Proxy(obj, {
  get: (obj, propKey) => {
    console.log(' get 操作')
    return obj[propKey]
  },
  set: (obj, propKey, value) => {
    console.log('设置 set 操作')
    obj[propKey] = value
  }
})
console.log(proxy.a) // get 操作 // 1
console.log(proxy.b.a1) // get 操作 // 11
console.log(proxy.b.b1.a2) // get 操作 // 21

console.log('--proxy.a--')
proxy.a = 'a' // 设置 set 操作
console.log('--proxy.b.a1--')
proxy.b.a1 = 'a1'  // 不触发 set
```

从上面我们可以发现，嵌套再深，我们都可以通过get监听到属性的访问。但是set并不像get自带递归，所以我们想要实现响应式，就需要对嵌套的对象或者数组，再次进行响应式处理。实现如下：

```js
let obj = {
  a: 1,
  b: {
    a1: 11,
    b1: {
      a2: 21
    }
  }
}
const handler = {
  get(obj, prop) {
    console.log(' get 操作')
	  const val = obj[prop]
    if(val !== null && typeof val=== 'object'){
      return new Proxy(val, handler);//代理内层
    }else{
      return val; // 返回obj[prop]
    }
  },
  set(obj, prop, value) {
    console.log('set', prop)
    obj[prop] = value
    return true // set需要返回true 代表赋值完成，否则会报错
  }
}
const proxy = new Proxy(obj, handler)
console.log(proxy.a) // get 操作 // 1
console.log(proxy.b.a1) // get 操作 // 11
console.log(proxy.b.b1.a2) // get 操作 // 21

console.log('--proxy.a--')
proxy.a = 'a' // set a
console.log('--proxy.b.a1--')
proxy.b.a1 = 'a1'  // set a1
```


除了get和set，proxy可以拦截多达13种操作，比如：`has(target, propKey)` ，可以拦截propKey in proxy 的操作，返回一个布尔值。

```js
// 使用 has 方法隐藏某些属性，不被 in 运算符发现
let handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false
    }
    return key in target;
  }
};
let target = { _prop: 'foo', prop: 'foo' };
let proxy = new Proxy(target, handler);
console.log('_prop' in proxy); // false
console.log('prop' in proxy); // true
```

## 性能的比较


Vue3在响应式的实现性能上做了优化，其中数据响应式的实现由`Object.defineProperty` API 改成了`Proxy` API。因此可能会有人认为`Proxy` API 的性能要优于 `Object.defineProperty`，但是实际上`Proxy`在性能上要比`Object.defineProperty`差的。具体的可以参考这篇文章：[Thoughts on ES6 Proxies Performance](https://thecodebarbarian.com/thoughts-on-es6-proxies-performance)。

若对象内部属性要全部递归代理，Proxy 可以只在调用的时候递归，而 Object.definePropery 需要一次完成所有递归，性能比 Proxy 差。 

在Vue2响应式实现中，definePropery 是在一开始，将传入的对象的所有属性全部进行递归，之后才处理set和get。 但是Vue3中的Proxy的递归是在set中，这样我们就可以根据需求来调整递归原则。也就是说，在一些条件下，让其不进行递归。这才是Vue3在响应式性能上优于Vue2的主要原因之一。

## 两者区别

- Proxy 是对整个对象的代理，而 Object.defineProperty 只能代理某个属性。在实现响应式函数的时候，defineProperty 需要对每个属性进行遍历添加代理。
- 对象上新增属性，Proxy可以监听到，Object.defineProperty不能。
- 数组新增修改，Proxy可以监听到，Object.defineProperty不能。
- 若对象内部属性要全部递归代理，Proxy可以只在调用的时候递归，而 Object.definePropery 需要一次完成所有递归，性能比 Proxy 差。
- Proxy 不兼容 IE，Object.defineProperty 不兼容 IE8 及以下。


## 参考资料

[ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/proxy)

[MDN-Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

> 如果你觉得这篇内容对你有帮助的话：
>
> 1、点赞支持下吧，让更多的人也能看到这篇内容
> 
> 2、公众号：前端极客技术，我们一起学习一起进步。