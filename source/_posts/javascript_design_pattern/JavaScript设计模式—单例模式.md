---
title: JavaScript设计模式—单例模式
urlname: javascript-singleton-pattern
tags:
  - 设计模式
  - 单例模式
categories:
  - 设计模式
abbrlink: 47019
date: 2021-01-17 12:16:40
---
在上一篇文章[《JavaScript设计模式——工厂模式》](https://juejin.cn/post/6916894964110589966)中我们一起学习了工厂模式，接下来我们一起来学习另一种设计模式——单例模式。

## 定义

单例模式：保证一个类只有一个实例，并提供一个访问它的全局访问点。无论创建多少次，都只返回第一次所创建的那唯一的一个实例。

单例模式是创建型设计模式的一种。针对全局仅需一个对象的场景。


> 欢迎关注我的微信公众号：**前端极客技术(FrontGeek)**

## 实现思路

在JavaScript中，我们如何才能保证一个类只有一个实例？

正常情况下，我们创建了一个类（本质上是构造函数），可以通过`new`关键字调用构造函数进而生成任意多个实例对象，例如：

```js
class SingleLoading {
  show () {
    console.log('这是一个单例Loading')
  }
}

let loading1 = new SingleLoading()
let loading2 = new SingleLoading()
console.log(loading1 === loading2) // false
```
上述代码中，我们先后new了loading1和loading2两个实例对象，两者是相互独立的对象，各占一块内存空间。

而单例模式想要做的，**是不论我们创建多少次，它都只返回第一次创建的那唯一一个实例给你**。

要实现上面的这一点，就需要**构造函数具备判断自己是否已经创建过实例的能力**。

现在我们将判断逻辑写成一个静态方法或直接写入构造函数的函数体内。

## 实现方式
单例模式的实现方式主要有两种：静态方法和闭包。

### 静态方法实现
下面我们用静态方法将上面的例子改造成单例模式：

```js
// 静态方法的实现
class SingleLoading {
  show () {
    console.log('这是一个单例Loading')
  }
  static getInstance(){
    // 判断是否已经创建过实例
    if (!SingleLoading.instance) {
      // 将创建的实例对象保持下来
      SingleLoading.instance = new SingleLoading()
    }
    return SingleLoading.instance
  }
}
const loading1 = SingleLoading.getInstance()
const loading2 = SingleLoading.getInstance()
console.log(loading1 === loading2) // true
```
上面代码中有一个`static`关键字，在getInstance方法前加上static，表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。


### 闭包
getInstance的逻辑还可以用闭包的方式实现：

```js
// 闭包
// 闭包
class SingleLoading {
  show () {
    console.log('这是一个单例Loading')
  }
}
SingleLoading.getInstance = (function(){
  // 定义自由变量instance，模拟私有变量
  let instance = null

  return function(){
    if(!instance) {
       // 如果为null则new出唯一实例
      instance = new SingleLoading()
    }
    return instance
  }
})();
const loading3 = new SingleLoading().getInstance()
const loading4 = new SingleLoading().getInstance()
console.log(loading3 === loading4)
```
借助闭包，在内存中保留了 instance 变量，不会被垃圾回收，用来保存唯一的实例，多次调用 new 的时候，只返回第一次创建的实例。


## 真题练习
我们已经学习了用静态方法和闭包来实现单例模式，接下来我们通过一道经典的面试题来巩固。

> 实现一个全局唯一的模态框（Modal弹框）

代码实现如下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>单例模式弹框</title>
</head>
<style>
  #modal {
    height: 200px;
    width: 200px;
    line-height: 200px;
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    border: 1px solid #999;
    text-align: center;
  }
</style>
<body>
  <button id="open">打开弹窗</button>
  <button id="close">关闭弹窗</button>
</body>
<script>
  const Modal = (function(){
    let modal = null;
    return function() {
      if (!modal) {
        modal = document.createElement('div')
        modal.innerHTML = '全局唯一的modal弹窗'
        modal.id = 'modal'
        modal.style.display = 'none'
        document.body.appendChild(modal)
      }
      return modal
    }
  })()

  document.getElementById('open').addEventListener('click', function (){
    const modal = new Modal()
    modal.style.display = 'block'
  })
  document.getElementById('close').addEventListener('click', function (){
    const modal = new Modal()
    modal.style.display = 'none'
  })
</script>
</html>
```
上面采用的是闭包的方法实现的，你也可以自己尝试用静态方法来实现。

## 总结

单例模式的核心：确保一个类只有一个实例。

对于单例模式的实现，如果采用class来实现，记住getInstance静态方法；如果采用闭包来实现，记住instance变量。

在许多优秀的前端库里，我们都能看到单例模式的身影。比如：Vuex和Redux这两个状态管理的库，它们都实现了一个全局的Store用于存储应用的所有状态。这个Store的实现，就是单例模式的典型应用。感兴趣的可以自己下载相应的源码研究一下。

最后，总结一下单例模式的优缺点：
- 优点：适用于单一对象，只生成一个对象实例，避免频繁创建和销毁实例，减少内存占用。
- 缺点：不适用动态扩展对象，或需创建多个相似对象的场景。


> 文章示例代码见：[singleton-pattern](https://github.com/Hanpeng-Chen/html-js-demo-code/tree/main/design-pattern/singleton-pattern)

## 参考文章
- [JavaScript 设计模式（一）：单例模式](https://juejin.cn/post/6844903874210299912#heading-6)
- [JavaScript 设计模式核⼼原理与应⽤实践](https://juejin.cn/book/6844733790204461070/section/6844733790267375624)


> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术(FrontGeek)**，我们一起学习一起进步。

### 实现一个Storage
> 实现Storage，使得该对象为单例，基于 localStorage 进行封装。实现方法 setItem(key,value) 和 getItem(key)。

```js
// 静态方法
class Storage {
  static getInstance (){
    if (!Storage.instance) {
      Storage.instance = new Storage()
    }
    return Storage.instance
  }
  getItem(key) {
    return localStorage.getItem(key)
  }
  setItem(key, value) {
    return localStorage.setItem(key, value)
  }
}


// 闭包
// 先实现一个基础的StorageBase类，将getItem和setItem放在它的原型上
function StorageBase(){}
StorageBase.prototype.getItem = function (key) {
  return localStorage.getItem(key)
}
StorageBase.prototype.setItem = function (key, value) {
  return localStorage.setItem(key, value)
}
// 以闭包的形式创建一个引用自由变量的构造函数
const Storage = (function(){
  let instance = null;
  return function(){
    if(!instance) {
      instance = new StorageBase()
    }
    return instance
  }
})()
const s1 = new Storage()
const s2 = new Storage()
s1.setItem('name', 'zhangsan')
s1.getItem('name')
s2.getItem('name')
console.log(s1 === s2)
```


## Vuex中的单例模式
Vuex：实现了一个全局的store用来存储应用的所有状态。这个store的实现就是单例模式的典型应用。

#### Vuex中如何确保store的唯一性

项目引入vuex的方式：
```js
// 安装插件
Vue.use(Vuex)

// 将store注入到vue实例
new Vue({
  el: '#app',
  store
})
```

vuex内部的install方法：
```js
let Vue // 这个Vue的作用和楼上的instance作用一样
...

export function install (_Vue) {
  // 判断传入的Vue实例对象是否已经被install过Vuex插件（是否有了唯一的state）
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  // 若没有，则为这个Vue实例对象install一个唯一的Vuex
  Vue = _Vue
  // 将Vuex的初始化逻辑写进Vue的钩子函数里
  applyMixin(Vue)
}
```

如果vuex的install没有实现单例，假如中途重新install，会为当前的Vue实例重新注入一个新的store，前面操作的数据全部没了。