---
title: 【面试题】Vue2 中如何检测数组变化
urlname: how-to-observe-array-in-vue2
tags:
  - vue
  - 面试
categories:
  - [前端, 面试]
  - [面试]
abbrlink: 17684
date: 2021-02-04 22:36:47
---


## 为什么要对数组进行单独处理
我们都知道在Vue2中，对响应式处理利用的是 `Object.defineProperty` 对数据进行拦截。如果数据是数组，我们还是用defineProperty的方法进行拦截的话，需要对数组每一层每一位都进行依赖收集和更新时的notify，这样消耗的性能将指数倍增加，因为我们无法预估一个数组有多少层有多少位。

考虑性能原因，没有用 defineProperty 对数组的每一项进行拦截，而是选择对下面7个数组方法进行重写：
- push
- shift
- pop
- splice
- unshift
- sort
- reverse

为什么是上面7个方法？因为我们要监控的是原数组的数据变化，使用上面7个方法对数组进行操作，会改变原数组，而其他方法并不会改变原数组。


## 如何重写数组方法
我们先来看Vue2中关于重写数组方法的源码：

```js
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]

/**
 * Intercept mutating methods and emit events
 */
methodsToPatch.forEach(function (method) { // 重写7个方法
  // cache original method
  // 获取数组原方法
  const original = arrayProto[method]
  // def方法重新定义arrayMethods的method方法，然后将新的取值方法赋值
  def(arrayMethods, method, function mutator (...args) {
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    // 如果是去除数组参数方法，触发一次notify将会重新计算
    // 如果仅仅是数字数据，任何操作只需要再次执行一次notify则可以
    // 但是如果新增的是一个对象类型，就需要重新监听
    // 为什么用角标和length属性不能监听的原因是因为无法触发obj的get方法，所以没法动态监听
    if (inserted) ob.observeArray(inserted)

    // notify change
    ob.dep.notify() // {a,b,c}   observer => dep
    return result
  })
})



/**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }

```

从上面的代码我们可以知道：vue2对push、splice等方法进行重写，如果加入新对象，会调用observe方法对新对象进行劫持。最后在向外抛出数组变化前，通知观察者进行视图更新。


## 补充

- 在Vue2中只有通过上面说的7中变异方法修改数组才会触发数组对应的watcher进行更新，如果我们修改数组的索引和长度是无法监控到的，也就无法实现视图更新。如果想更改索引更新数据，可以通过 `Vue.$set()` 来进行处理。

- 数组中数据如果是对象类型，也会进行递归劫持。

> 欢迎关注我的微信公众号：**前端极客技术(FrontGeek)**

> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术(FrontGeek)**，我们一起学习一起进步。