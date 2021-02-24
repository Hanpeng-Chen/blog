---
title: 这一次彻底掌握JavaScript的深浅拷贝
urlname: javascript-shallow-copy-and-deep-copy
tags:
  - 深浅拷贝
categories:
  - 前端
  - JavaScript
abbrlink: 42023
date: 2021-02-22 23:30:45
---
关于拷贝这个问题，也是前端面试中的一道经典面试题，我们在日常开发中也常碰到需要用到深拷贝或浅拷贝的场景。接下来我们通过这篇文章，彻底掌握JavaScript的深浅拷贝。

## 数据类型
在开始讲深浅拷贝之前，我们要先知道JavaScript的数据类型，主要有下图所示的8种：

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/20210222225118.png)

Object是引用类型，其他7种为基础类型。

JavaScript的数据类型最后都会在初始化之后放在不同的内存中，因此上面的数据类型大致可以分为两类来进行存储：
- 基础类型存储在栈内存，被引用或拷贝时，会创建一个完全相等的变量
- 引用类型存储在堆内存，存储的是地址，多个引用指向同一个地址，这里会涉及一个“共享”的概念。

接下来我们先来看看浅拷贝。

> 欢迎关注公众号：**前端极客技术(FrontGeek)**

## 浅拷贝的原理和实现

浅拷贝的定义：

> 创建一个对象接受要重新复制或引用的对象值。如果对象属性是基本的数据类型，复制的就是基本类型的值给新对象；如果属性是引用数据类型，复制的就是内存中的地址，如果其中一个对象改变了这个内存中的地址，可能会影响到另一个对象。


我们来看看JavaScript中都有哪些实现浅拷贝的方法。

### Object.assign
object.assign 是 ES6 中 object 的一个方法，该方法可以用于 JS 对象的合并等多个用途，其中一个用途就是可以进行浅拷贝。该方法的第一个参数是拷贝的目标对象，后面的参数是拷贝的来源对象（也可以是多个来源）。

> object.assign 的语法为：Object.assign(target, ...sources)

object.assign 的示例代码如下：

```js
let target = {}
let source = {
  a: {
    b: 1
  }
}
Object.assign(target, source)
source.a.b = 10; 
console.log(source); // { a: { b: 10 } }; 
console.log(target); // { a: { b: 10 } };
```

从上面代码中我们可以看到，首先通过 Object.assign 将 source 拷贝到 target 对象中，将 source 对象中的 b 属性由 1 修改为 10。从执行结果可以看出target和source两个对象中的 b 属性都变为 10 了，证明 Object.assign 暂时实现了我们想要的拷贝效果。

使用Object.assign方法实现浅拷贝时有几点需要注意：
- Object.assign方法不会拷贝对象的继承属性
- Object.assign方法不会拷贝对象的不可枚举的属性
- 可以拷贝Symbol类型的属性


```js
let obj1 = { a: { b: 1 }, sym: Symbol(1) };

Object.defineProperty(obj1, "innumerable", {
  value: "不可枚举属性",

  enumerable: false,
});

let obj2 = {};
Object.assign(obj2, obj1);
obj1.a.b = 10;
console.log("obj1", obj1); // obj1 { a: { b: 10 }, sym: Symbol(1) }
console.log("obj2", obj2); // obj1 { a: { b: 10 }, sym: Symbol(1) }

```
从上面的样例代码中可以看到，利用 object.assign 也可以拷贝 Symbol 类型的对象，但是如果到了对象的第二层属性 obj1.a.b 这里的时候，前者值的改变也会影响后者的第二层属性的值，说明其中依旧存在着访问共同堆内存的问题，也就是说这种方法还不能进一步复制，而只是完成了浅拷贝的功能。


### 扩展运算符方法
我们也可以利用JS的扩展运算符，在构造对象的同时完成浅拷贝的功能。

```js
let obj = {a: 1, b: {c: 10}}
let obj2 = {...obj}
obj2.a = 10
console.log(obj);
console.log(obj2);

let arr = [1, 3, 4, 5, [10, 20]]
let arr2 = [...arr]
console.log(arr2)
```
扩展运算符 和 object.assign 有同样的缺陷，也就是实现的浅拷贝的功能差不多，但是如果属性都是基本类型的值，使用扩展运算符进行浅拷贝会更加方便。

### concat拷贝数组
数组的concat方法也是浅拷贝，所以连接一个含有引用类型的数组时，需要注意修改原数组中元素的属性，因为它会影响拷贝之后连接的数组。

```js
let arr = [1, 2, 3];
let newArr = arr.concat();
newArr[1]= 10;
console.log(arr);
console.log(newArr);
```

### slice拷贝数组
slice方法也比较有局限性，因为它仅仅针对数组类型。

slice方法会返回一个新的数组对象，这一对象由该方法的前两个参数来决定原数组截取的开始和结束时间，是不会影响和改变原数组的。

```js
let arr = [1, 2, {val: 4}];
let newArr = arr.slice();
newArr[2].val = 1000;
console.log(arr);  //[ 1, 2, { val: 1000 } ]
```

从上面的代码中可以看出，这就是浅拷贝的限制所在了——它只能拷贝一层对象。如果存在对象的嵌套，那么浅拷贝将无能为力。因此深拷贝就是为了解决这个问题而生的，它能解决多层对象嵌套问题，彻底实现拷贝。这一讲的后面我会介绍深拷贝相关的内容。

### 手工实现一个浅拷贝

除了上面提到的方法，我们也可以手写一个浅拷贝方法，思路如下：
- 对基础数据类型做一个最基本的拷贝；
- 对引用类型开辟一个新的存储，并拷贝一层对象属性。

```js
const shallowCopy = (target) => {
  if (typeof target === "object" && target !== null) {
    const copyTarget = Array.isArray(target) ? [] : {};
    for (let key in target) {
      if (target.hasOwnProperty(key)) {
        copyTarget[key] = target[key];
      }
    }
    return copyTarget;
  } else {
    return target;
  }
};
```


## 深拷贝的原理和实现

浅拷贝只是创建了一个新的对象，复制了原有对象的基本类型的值，而引用数据类型只拷贝了一层属性，再深层的还是无法进行拷贝。深拷贝则不同，对于复杂引用数据类型，其在堆内存中完全开辟了一块内存地址，并将原有的对象完全复制过来存放。

这两个对象是相互独立、不受影响的，彻底实现了内存上的分离。总的来说，深拷贝的原理可以总结如下：

> 将一个对象从内存中完整地拷贝出来一份给目标对象，并从堆内存中开辟一个全新的空间存放新对象，且新对象的修改并不完全改变源对象，二者实现真正的分离。

知道了深拷贝的原理后，我们来看下都有哪些深拷贝的方法：

### 乞丐版：JSON.stringify

JSON.stringify() 方法是开发中常用的也是最简单的深拷贝方法，其实就是将一个对象序列化为JSON的字符串，并将对象中的内容转换为字符串，最后再用JSON.parse()方法将JSON字符串生成一个新的对象。

```js
let obj = {a: 1, b: [1, 2, 3]};
let str = JSON.stringify(obj);
let newObj = JSON.parse(str);

obj.a = 2;
obj.b.push(4)
console.log(obj)
console.log(newObj)
```
从上面的代码可以看到，通过 JSON.stringify 可以初步实现一个对象的深拷贝，通过改变 obj 的 a 属性，其实可以看出 newObj 这个对象也不受影响。


我们来看看下面几种使用JSON.stringify方法进行深拷贝的情况：
```js
function Obj() { 
  this.func = function () { alert(1) }; 
  this.obj = {a:1};
  this.arr = [1,2,3];
  this.und = undefined; 
  this.reg = /123/; 
  this.date = new Date(0); 
  this.NaN = NaN;
  this.infinity = Infinity;
  this.sym = Symbol(1);
} 

let obj1 = new Obj();
Object.defineProperty(obj1,'innumerable',{ 
  enumerable:false,
  value:'innumerable'
});

console.log('obj1',obj1);
let str = JSON.stringify(obj1);
let obj2 = JSON.parse(str);
console.log('obj2',obj2);
```

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/20210222231521.png)

从上面的执行结果来看，JSON.stringify方法来实现深拷贝并不完美，下面我们一起来看看JSON.stringify实现深拷贝存在的问题：

- 拷贝的对象如果有函数、undefined、symbol这几种类型，经过JSON.stringify序列化之后的字符串中这个键值对会消失；
- 拷贝Date引用类型会变成字符串
- 无法拷贝不可枚举的属性
- 无法拷贝对象的原型链
- 拷贝RegExp引用类型会变成空对象
- 对象中含有 NaN、Infinity、-Infinity，JSON序列化的结果会变成null
- 无法拷贝对象的循环应用，即对象成环，例如`obj[key]=obj`


用 JSON.stringify 方法实现深拷贝对象，虽然到目前为止还有很多无法实现的功能，但是这种方法足以满足日常的开发需求，并且是最简单和快捷的。

如果深拷贝方法要支持Function、Symbol等类型的拷贝，我们只能自己手动实现一个深拷贝方法。


### 手写实现深拷贝（基础版）
接下来我们自己实现一个深拷贝方法，思路如下：

> 通过for in遍历传入参数的属性值，如果值时引用类型则再次递归调用该方法，如果时基础类型就直接复制。

```js
function deepCopy(target) {
  let copyTarget = Array.isArray(target) ? [] : {};
  for (let key in target) {
    if (typeof target[key] === "object") {
      copyTarget[key] = deepCopy(target[key]);
    } else {
      copyTarget[key] = target[key];
    }
  }
  return copyTarget;
}

let obj1 = {
  a: {
    b: 1,
  },
  c: 10,
};
let obj2 = deepCopy(obj1);
obj1.a.b = 100;
obj1.c = 22

console.log(obj2);
```

上面我们通过递归的方式实现了深拷贝，但仍存在一些问题没有解决：
- 无法复制不可枚举的属性以及Symbol类型；
- 只针对普通的引用类型的值做了递归复制，但对于Array、Date、RegExp、Error、Function这些引用类型并不能正确地拷贝。
- 对象的属性里面成环，即循环引用没有解决。

接下来我们一起来改进这个深拷贝方法：

### 改进版的深拷贝方法
针对上面说的几点问题，一起来看看有什么解决方法：
- 针对不可枚举属性和Symbol类型，可以使用 `Reflect.ownKeys` 方法；
- 参数为Date、RegExp类型时，直接生成一个新的实例返回；
- 利用 Object 的 getOwnPropertyDescriptors 方法可以获得对象的所有属性，以及对应的特性，顺便结合 Object.create() 方法创建一个新对象，并继承传入原对象的原型链；
- 利用 WeakMap 类型作为 Hash表，因为WeakMap时弱引用类型，可以有效防止内存泄漏，作为检测循环引用很有帮助，如果存在循环，则引用直接返回WeakMap存储的值。


根据上面的内容，我们重新写一下深拷贝方法：
```js
function deepCopy(obj, hash = new WeakMap()) {
  // 日期对象直接返回一个新的日期对象
  if (obj.constructor === Date) return new Date(obj);
  // 如果是正则对象直接返回一个新的正则对象
  if (obj.constructor === RegExp) return new RegExp(obj);
  // 如果循环引用了就用WeakMap来解决
  if (hash.has(obj)) return hash.get(obj);

  // 遍历传入参数所有键的特性
  let allDesc = Object.getOwnPropertyDescriptor(obj);
  // 继承原型链
  let copyObj = Object.create(Object.getPrototypeOf(obj), allDesc);
  hash.set(obj, copyObj)
  for (let key of Reflect.ownKeys(obj)) {
    copyObj[key] = (isComplexDataType(obj[key]) && typeof obj[key] !== 'function') ? deepCopy(obj[key], hash) : obj[key]
  }
  return copyObj
}


function isComplexDataType(obj) {
  return (typeof obj === 'object' || typeof obj === 'function') && obj !== null
}


// 下面是验证代码
let obj = {
  num: 0,
  str: '',
  boolean: true,
  unf: undefined,
  nul: null,
  obj: { name: '我是一个对象', id: 1 },
  arr: [0, 1, 2],
  func: function () { console.log('我是一个函数') },
  date: new Date(0),
  reg: new RegExp('/我是一个正则/ig'),
  [Symbol('1')]: 1,
};
Object.defineProperty(obj, 'innumerable', {
  enumerable: false, value: '不可枚举属性' }
);
obj = Object.create(obj, Object.getOwnPropertyDescriptors(obj))
obj.loop = obj    // 设置loop成循环引用的属性
let cloneObj = deepCopy(obj)
cloneObj.arr.push(4)
console.log('obj', obj)
console.log('cloneObj', cloneObj)
```

### 性能问题
尽管使用深拷贝可以完全克隆一个新的对象，不会产生副作用，但是因为深拷贝方法用到递归，性能方面不如浅拷贝，在实际开发过程中，我们要根据实际情况进行选择。

> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术(FrontGeek)**，我们一起学习一起进步。


![](https://gitee.com/HanpengChen/blog-images/raw/master/%E5%89%8D%E7%AB%AF%E6%9E%81%E5%AE%A2%E6%8A%80%E6%9C%AF%E4%BA%8C%E7%BB%B4%E7%A0%81.png)