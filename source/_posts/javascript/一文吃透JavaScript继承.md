---
title: 「面试必备」一文吃透JavaScript继承
urlname: inheritance-in-javascript
tags:
  - 继承
categories:
  - [前端, JavaScript]
abbrlink: 20247
date: 2021-02-28 23:53:15
---
继承在各种编程语言中都充当着至关重要的角色，在JavaScript中也被经常用在前端工程基础库的底层搭建上，是JavaScript需要重点学习的一块内容。

继承可以使得子类具有父类的各种方法和属性。ES6中推出了class这个概念，方便了我们学习和理解，但class只是一个语法糖，实际底层的实现还是原来的那一套：利用原型链和构造函数来实现继承，接下来我们一起来看看在JavaScript中都有哪些实现继承的方法。

## 原型链继承
原型链继承是实现继承的主要方法，基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

该方法主要涉及构造函数、原型和实例，三者之间存在着一定的关系：

> 每一个构造函数都有一个prototype属性，指向函数的原型对象；每一个原型对象都有一个constructor属性，指向构造函数；每一个实例都有一个 __proto__ 属性，指向构造函数的原型对象。

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210228225313.png)

我们先来看下面一段代码：
```js
function Parent() {
  this.name = "parent";
  this.interest = ["eat"];
}

function Child() {
  this.type = "child";
}

Child.prototype = new Parent();
let child = new Child();
console.log(child.name, child.type);
```

在上面的代码中，定义了 Parent 和 Child 两个对象，两个对象之间实现了继承，这种继承方式是通过创建Parent的实例，并将该实例赋给 Child.prototype 实现的。该方法实现的本质就是重写了原型对象。

从上面的代码看，在子类的实例中可以访问到父类的属性和方法，看似这种继承方式没什么问题，我们接着看下面的代码：
```js
let child1 = new Child();
child1.interest.push("run");
console.log(child.interest, child1.interest); // [ 'eat', 'run' ] [ 'eat', 'run' ]
```

上面代码中我又新创建了一个子类实例 `child1`，并改变了 `interest` 属性，但是原来的 `child` 的interest属性也跟着变了。

出现这个问题的原因很简单：因为两个实例使用的是同一个原型对象，它们的内存空间是共享的。当一个发生变化时，另一个也随之变化，这就是使用原型链继承方式的一个缺点。

还有一个缺点就是：没有办法在不影响所有对象实例的情况下，给父类的构造函数传递参数。

因为上面的问题，实践中很少会单独使用原型链继承。


## 构造函数继承
为了解决原型属性共享问题，开发人员开始使用一种叫做借用构造函数（constructor stealing）的技术，有时候也叫伪造对象或经典继承。

借助构造函数的基本思想就是：利用call或apply把父类中通过this指定的属性和方法复制到子类创建的实例中。因为this对象是在运行时基于函数的执行环境绑定的。

我们通过下面代码来了解：
```js
function Parent(name) {
  this.name = name;
  this.interest = ["eat"];
}
Parent.prototype.getName = function () {
  return this.name;
};

function Child(name) {
  Parent.call(this, name);
  this.type = "child";
}

let child1 = new Child('child1');
child1.interest.push("run");
console.log(child1.interest); // [ 'eat', 'run' ]
let child2 = new Child('child2');
console.log(child2.interest); // [ 'eat' ]

console.log(child1.getName()); // 报错
```

从上面代码的执行结果来看，该方法解决了原型链继承的弊端，但仍存在问题：父类原型对象上一旦存在父类之前自己定义的方法，子类将无法继承这些方法。

我们可以总结出构造函数实现继承具有如下优缺点：
**优点：**
- 它使父类的引用类型属性不会被共享；
- 可以向父类的构造函数传参。

**缺点：**
- 只能继承父类的实例属性和方法，不能继承原型属性或方法。


## 组合继承（原型链+构造函数）
组合继承（combination inheritance），也叫作伪经典继承。是将原型链和借用构造函数的技术组合在一起，发挥二者之长的一种继承方式。

组合继承方法的思路是将公共的属性和方法放在父类的 prototype 上，然后利用原型链继承来实现公共的属性和方法的继承，而对于那种每个实例都可自定义修改的属性采取构造函数继承的方法来实现每个实例都独有一份这样的属性。

代码如下：
```js
function Parent() {
  this.name = "parent";
  this.interest = ["eat"];
}

Parent.prototype.getName = function () {
  return this.name;
};

function Child() {
  Parent.call(this);
  this.type = "child type";
}

Child.prototype = new Parent();
Child.prototype.constructor = Child;

let child1 = new Child();
let child2 = new Child();

child1.interest.push("run");
console.log(child1.interest, child2.interest); // [ 'eat', 'run' ] [ 'eat' ]
console.log(child1.getName()); // parent
console.log(child2.getName()); // parent
```

从代码执行结果来看，组合继承避免了原型链继承和构造函数继承的缺陷，融合了二者的优点。

但组合继承有一个问题：就是无论什么情况下，都会调用两次超类型的构造函数，即Parent执行了两次，第一次是改变 Child的prototype的时候，第二次是通过call方法调用Parent的时候。

上面介绍的三种方法主要是围绕构造函数的方式，如果是JavaScript的普通对象，要如何实现继承？

## 原型式继承
该方法的原理就是借助原型，可以基于已有的对象创建新对象，节省了创建自定义类型这一步。
```js
function object(o) {
  function W(){
  }
  W.prototype = o;
  return new W();
}
```

ES5中新增了 Object.create() 方法规范化了原型式继承，这个方法接收两个参数：一是用作新对象原型的对象、二是为新对象定义额外属性的对象（可选参数）。

通过下面的代码我们一起看看普通对象时怎样实现继承的。
```js
let parent = {
  name: "parent",
  interest: ["eat"],
  getName: function () {
    return this.name;
  },
};

let parent1 = Object.create(parent);
parent1.name = "parent1";
parent1.interest.push("sleep");

let parent2 = Object.create(parent);
parent2.name = "parent2";
parent2.interest.push("run");

console.log(parent1.name); // parent1
console.log(parent1.name === parent1.getName()); // true
console.log(parent1.interest); // [ 'eat', 'sleep', 'run' ]
console.log(parent2.name); // parent
console.log(parent2.name === parent2.getName()); // true
console.log(parent2.interest); // [ 'eat', 'sleep', 'run' ]
```

从上面代码执行结果你会发现存在引用类型数据共享问题，因为Object.create方法是可以为一些对象实现浅拷贝的。

关于浅拷贝的内容可以看我之前写的文章：[这一次，彻底掌握JavaScript的深浅拷贝](https://juejin.cn/post/6932310945088290830/)

原型式继承的缺点：多个实例的引用类型属性指向相同的内存，存在篡改的可能。

接下来我们看下在原型式继承基础上进行优化的另一种继承方式：寄生式继承。

## 寄生式继承

寄生式继承时原型式继承的加强版，利用原型式继承可以获得一份目标对象的浅拷贝的能力再进行增强，添加一些方法，这样的继承方式称为寄生式继承。

寄生式继承的优缺点和原型式继承一样，但对于普通对象的继承方式来说，寄生式继承相比于原型式继承，在父类的基础上添加了更多的方法。下面我们一起来看下其实现代码：

```js
let parent = {
  name: "parent",
  interest: ["eat", "run"],
  getName: function () {
    return this.name;
  },
};

function clone(original) {
  let clone = Object.create(original);
  clone.getInterest = function () {
    return this.interest;
  };
  return clone;
}

let parent1 = clone(parent);
console.log(parent1.getName()); // parent
console.log(parent1.getInterest()); // [ 'eat', 'run' ]
```

从上面代码可以看到，parent1是通过寄生式继承生成的实例，不仅有getName方法，还拥有getInterest方法。


## 寄生组合式继承
实质上，寄生组合继承是寄生式继承的加强版。这是为了避免组合继承中无可避免地要调用两次父类构造函数的最佳方案，也是所有继承方式中相对最优的继承方式。

代码如下：
```js
function clone(parent, child) {
  // 改用Object.create可以减少组合继承中多进行一次构造函数
  child.prototype = Object.create(parent.prototype);
  child.prototype.constructor = child;
}

function Parent() {
  this.name = "parent";
  this.interest = ["eat", "run"];
}

Parent.prototype.getName = function () {
  return this.name;
};

function Child() {
  Parent.call(this);
  this.type = "child type";
}

clone(Parent, Child);

Child.prototype.getInterest = function () {
  return this.interest;
};

let child1 = new Child();
let child2 = new Child();

child1.interest.push("sleep");
console.log(child1.getName()); // parent
console.log(child1.getInterest()); // [ 'eat', 'run', 'sleep' ]
console.log(child2.getName()); // parent
console.log(child2.getInterest()); // [ 'eat', 'run' ]
```
通过这段代码可以看出来，这种寄生组合式继承方式，基本可以解决前几种继承方式的缺点，较好地实现了继承想要的结果，同时也减少了构造次数，减少了性能的开销。


ES6新增了Class语法糖，并提供了继承的关键字extends，接下来我们看下extends的用法和底层实现逻辑。

## ES6的Class继承
Class可以通过extends关键字实现继承。子类必须在constructor方法中调用super方法，否则新建实例时会报错。这是因为子类自己的this对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用super方法，子类就得不到this对象。

大多数浏览器的ES5实现中，每一个对象都有 __proto__ 属性，指向对应的构造函数的prototype属性。Class作为构造函数的语法糖，同时有prototype和__proto__两个属性，因此同时存在两条继承链:

- 子类的__proto__属性，表示构造函数的继承，总是指向父类；
- 子类的prototype属性的__proto__属性，表示方法的继承，总是指向父类的prototype属性。

```js
class Parent {
  constructor(name) {
    this.name = name;
  }
  getName() {
    return this.name;
  }
}

class Child extends Parent {
  constructor(name, age) {
    super(name);
    this.age = age;
  }
  getAge() {
    return this.age;
  }
}

let child = new Child("zhangsan", 25);
console.log(child.getName());
console.log(child.getAge());

console.log(Child.__proto__ === Parent); // true
console.log(Child.prototype.__proto__ === Parent.prototype); // true
```

在实际项目开发过程中，因为浏览器兼容性问题，我们都会利用babel将ES6的代码编译成ES5。那接下来我们来看看extends编译成ES5语法是什么样子的，下面是转译的代码：

```js
"use strict";

function _typeof(obj) { "@babel/helpers - typeof"; if (typeof Symbol === "function" && typeof Symbol.iterator === "symbol") { _typeof = function _typeof(obj) { return typeof obj; }; } else { _typeof = function _typeof(obj) { return obj && typeof Symbol === "function" && obj.constructor === Symbol && obj !== Symbol.prototype ? "symbol" : typeof obj; }; } return _typeof(obj); }

function _instanceof(left, right) { if (right != null && typeof Symbol !== "undefined" && right[Symbol.hasInstance]) { return !!right[Symbol.hasInstance](left); } else { return left instanceof right; } }

function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function");
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: { value: subClass, writable: true, configurable: true }
  });
  if (superClass) _setPrototypeOf(subClass, superClass);
}

function _setPrototypeOf(o, p) { _setPrototypeOf = Object.setPrototypeOf || function _setPrototypeOf(o, p) { o.__proto__ = p; return o; }; return _setPrototypeOf(o, p); }

function _createSuper(Derived) { var hasNativeReflectConstruct = _isNativeReflectConstruct(); return function _createSuperInternal() { var Super = _getPrototypeOf(Derived), result; if (hasNativeReflectConstruct) { var NewTarget = _getPrototypeOf(this).constructor; result = Reflect.construct(Super, arguments, NewTarget); } else { result = Super.apply(this, arguments); } return _possibleConstructorReturn(this, result); }; }

function _possibleConstructorReturn(self, call) { if (call && (_typeof(call) === "object" || typeof call === "function")) { return call; } return _assertThisInitialized(self); }

function _assertThisInitialized(self) { if (self === void 0) { throw new ReferenceError("this hasn't been initialised - super() hasn't been called"); } return self; }

function _isNativeReflectConstruct() { if (typeof Reflect === "undefined" || !Reflect.construct) return false; if (Reflect.construct.sham) return false; if (typeof Proxy === "function") return true; try { Boolean.prototype.valueOf.call(Reflect.construct(Boolean, [], function () {})); return true; } catch (e) { return false; } }

function _getPrototypeOf(o) { _getPrototypeOf = Object.setPrototypeOf ? Object.getPrototypeOf : function _getPrototypeOf(o) { return o.__proto__ || Object.getPrototypeOf(o); }; return _getPrototypeOf(o); }

function _classCallCheck(instance, Constructor) { if (!_instanceof(instance, Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

function _defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } }

function _createClass(Constructor, protoProps, staticProps) { if (protoProps) _defineProperties(Constructor.prototype, protoProps); if (staticProps) _defineProperties(Constructor, staticProps); return Constructor; }

var Parent = /*#__PURE__*/function () {
  function Parent(name) {
    _classCallCheck(this, Parent);

    this.name = name;
  }

  _createClass(Parent, [{
    key: "getName",
    value: function getName() {
      return this.name;
    }
  }]);

  return Parent;
}();

var Child = /*#__PURE__*/function (_Parent) {
  _inherits(Child, _Parent);

  var _super = _createSuper(Child);

  function Child(name, age) {
    var _this;

    _classCallCheck(this, Child);

    _this = _super.call(this, name);
    _this.age = age;
    return _this;
  }

  _createClass(Child, [{
    key: "getAge",
    value: function getAge() {
      return this.age;
    }
  }]);

  return Child;
}(Parent);
```
从编译后的代码可以看到，采用的也是寄生组合式继承方式，这也证明了这种方式是较优的解决继承的方式。


> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术(FrontGeek)**，我们一起学习一起进步。