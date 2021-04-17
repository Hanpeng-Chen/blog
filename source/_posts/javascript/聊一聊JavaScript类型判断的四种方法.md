---
title: 聊一聊JavaScript类型判断的四种方法
urlname: javascript-methods-of-judging-data-type
tags:
  - JavaScript
  - 类型判断
categories:
  - [前端, JavaScript]
abbrlink: 64823
date: 2021-01-20 20:09:02
---
## 前言
在web开发中，我们经常碰到需要判断数据是数字还是字符串，判断是数组还是对象的场景，接下来我们一起来看看JavaScript中都有哪些方法可以判断数据类型。

> 欢迎关注我的微信公众号：**前端极客技术(FrontGeek)**

## typeof
在JS中，我们最常用的判断方法自然是`typeof`。

> typeof：是一元操作符，放在其单个操作数的前面，操作数可以是任意类型。返回值为表示操作数类型的一个字符串。

在ES5中，JavaScript有六种数据类型：Number、String、Boolean、Undefined、Null、Object。ES6新增了一种数据类型：Symbol。

我们先用typeof对上面七种数据类型的值进行运算，看得到的结果是什么样的：
```js
console.log(typeof 1) // number
console.log(typeof 'test') // string
console.log(typeof true) // boolean
console.log(typeof undefined) // undefined
console.log(typeof null) // object
console.log(typeof {}) // object
console.log(typeof []) // object
console.log(typeof Symbol()) // symbol
```

从执行结果我们可以看出，null、对象都返回`object`，其他的都返回和数据类型对应的小写字符串。

在JS中，Object类型下还细分很多类型，比如Array、Function、Date、Error等等，我们也用typeof尝试判断这几种类型：
```js

console.log(typeof []) // object
function a(){}
console.log(typeof a) // function
let now = new Date()
console.log(typeof now) // object
let error = new Error()
console.log(typeof error) // object
```

typeof可以检测出函数类型，其他类型返回的都是object，没办法确实是哪一种细分类型。


## instanceof
instanceof操作符用于检查一个对象是否属于某个特定的class。同时它还考虑了继承。

用法如下：
```js
A instanceof B
```
表示判断A是否为B的实例，如果是返回true，否则返回false。

我们先来看下几个关于instanceof的例子：
```js
console.log([] instanceof Array) // true
console.log([] instanceof Object) // true

console.log({} instanceof Object) // true

console.log(new Date() instanceof Date) // true

function a (){}
console.log(a instanceof Function) // true
console.log(new a() instanceof a) // true
```

从上面我们可以发现一个问题，instanceof虽然可以判断出[]是Array的实例，但它也认为 []是Object的实例。

我们简单分析下[]、Array、Object三者的关系：

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/20210117211231.png)

[].__proto__ 指向 Array.prototype，而 Array.prototype.__proto__ 又指向 Object.prototype，最终 Object.prototype.__proto__ 指向null，标志着原型链的结束。因此，[]、Array、Object 就在内部形成了一条原型链。

我们从原型链可以看出，[] 的 __proto__  直接指向Array.prototype，间接指向 Object.prototype，所以按照 instanceof 的判断规则，[] 就是Object的实例。依次类推，类似的 new Date()、new Person() 也会形成一条对应的原型链 。因此：

> instanceof 只能用来判断两个对象是否属于实例关系， 而不能判断一个对象实例具体属于哪种类型。


## constructor
对于对象子类型的判断，除了instanceof，我们还可以利用对象的constructor属性来进行判断。

constructor属性表示原型对象与构造函数之间的关联关系，如果修改了原型对象，一般会同时修改constructor属性，防止引用的时候出错。所以，修改原型对象时，一般要同时修改constructor属性的指向。

```js
console.log('22'.constructor === String)             // true
console.log(true.constructor === Boolean)            // true
console.log([].constructor === Array)                // true
console.log(new Number(22).constructor === Number)   // true
console.log(new Function().constructor === Function) // true
console.log((new Date()).constructor === Date)       // true
console.log(new RegExp().constructor === RegExp)     // true
console.log(new Error().constructor === Error)       // true
```

在使用constructor时我们需要注意以下几点：
- null 和 undefined 没有构造函数，因此不会有constructor的存在，这两种类型需要通过其他方式判断；
- 函数的constructor 是不稳定的，这个主要体现在自定义对象上，当开发者重写 prototype 后，原有的 constructor 引用会丢失，constructor 会默认为 Object


## Object.prototype.toString
toString()是Object的原型方法，调用该方法，默认返回当前对象的`[[Class]]`。`[[Class]]`是一个内部属性，其格式为 [object Xxx] ，其中 Xxx 就是对象的类型。

对于Object对象，直接调用toString方法就能返回`[object Object]`，而对于其他对象，需要通过call/apply来调用才能返回正确的类型信息。

我们来看个demo：
```js
console.log(Object.prototype.toString.call(''))   // [object String]
console.log(Object.prototype.toString.call(1))    // [object Number]
console.log(Object.prototype.toString.call(true)) // [object Boolean]
console.log(Object.prototype.toString.call(Symbol())) //[object Symbol]
console.log(Object.prototype.toString.call(undefined)) // [object Undefined]
console.log(Object.prototype.toString.call(null)) // [object Null]
console.log(Object.prototype.toString.call(new Function())) // [object Function]
console.log(Object.prototype.toString.call(new Date())) // [object Date]
console.log(Object.prototype.toString.call([])) // [object Array]
console.log(Object.prototype.toString.call(new RegExp())) // [object RegExp]
console.log(Object.prototype.toString.call(new Error())) // [object Error]
console.log(Object.prototype.toString.call({})) // [object Object]
console.log(Object.prototype.toString.call(Math)) // [object Math]
console.log(Object.prototype.toString.call(JSON)) // [object JSON]

function a () {
  console.log(Object.prototype.toString.call(arguments)) // [object Arguments]
}
a()
```

Object.prototype.toString 看起来就是一个神器，通用性强，但是相比其他几个方法来说比较繁琐，在实际开发过程中，我们可以对其封装成一个判断类型的函数进行调用。

## type API
在实际开发过程中，我们不会去检测 Math、JSON和Arguments，所以在isType函数中不考虑。

封装结果如下：
```js
function isType(value, type) {
  return Object.prototype.toString.call(value) === `[object ${type}]`
}
console.log(isType('111', 'String')) // true
console.log(isType(null, 'Null')) // true
```


## 总结
当你想判断一个基本类型的数据时，你可以用typeof去判断，它很简单，而且可靠；当你想判断一个对象属于哪个子类型时，你可以使用instanceof运算符或constructor属性，但是你需要有个预期的类型，不然就要针对每一种类型写不一样的if...else...语句，还有一点需要注意的就是constructor属性可以被修改，所以并不可靠；如果你不嫌代码量多，要求准确且全面，那你可以用Object.prototype.toString.call()进行判断。

文中所有示例代码见：[judge-type-of-data](https://github.com/Hanpeng-Chen/html-js-demo-code/tree/main/judge-type-of-data)



> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术(FrontGeek)**，我们一起学习一起进步。