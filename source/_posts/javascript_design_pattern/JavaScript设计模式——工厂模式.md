---
title: JavaScript设计模式——工厂模式
urlname: javascript-factory-pattern
tags:
  - 设计模式
  - 工厂模式
categories:
  - 设计模式
abbrlink: 5473
date: 2021-01-13 23:37:21
---
在瞬息万变的前端领域，技术更新迭代非常快，我们经常能在网络上看到诸如“学不动了”之类的言论。但是作为一名前端开发工程师，除了各种新技术，还有许多“一次学习，终身受益”的知识值得我们花时间去学习，设计模式就是其中之一。


## 设计模式
在学习设计模式之前，我们先要知道什么是设计模式。

我们先来看下维基百科上关于设计模式的定义：

> 在软件工程中，设计模式（design pattern）是对软件设计中普遍存在（反复出现）的各种问题，所提出的解决方案。

设计模式并不是一种固定的公式，而是一种思想，是一种解决问题的思路；恰当的使用设计模式，可以实现代码的复用和提高可维护性。


## SOLID设计原则
设计原则是设计模式的知道理论，可以帮助我们规避不良的软件设计。SOLID指代的五个基本原则分别是：

- 单一功能原则(Single Responsibiity Principle)
    - 一个程序只做好一件事
    - 如果功能过于复杂就拆分开，每个部分保持独立
- 开放封闭原则(Opened Closed Principle)
    - 对扩展开放，对修改封闭
    - 增加需求时，扩展新代码，而非修改已有代码
- 里氏替换原则(Liskov Substitution Principle)
    - 子类能覆盖父类
    - 父类能够出现的地方子类就能出现
- 接口隔离原则(Interface Segregation Principle)
    - 保持接口的单一独立
    - 类似单一职责原则，这里更关注接口
- 依赖反转原则(Dependency Inversion Principle)
    - 面向接口编程，依赖于抽象而不依赖于具体
    - 使用方只关注接口而不关注具体类的实现


Javascript设计模式中，主要用到的设计模式基本都是围绕单一功能和开放封闭两个原则展开的。

## 设计模式分类
设计模式有23种，可以按照创建型、行为型、结构型划分成三类，具体见下图：

![](https://image.chenhanpeng.com/static/blog-images/blogImages/2021/spring/20210112163348.png)

针对这23中设计模式，我们将选一些比较重要、实际开发中能用到、面试中常见的来详细学习。

> 欢迎关注我的微信公众号：**前端极客技术(FrontGeek)**

下面我们先来学习工厂模式：

## 工厂模式
工厂模式是用来创建对象的一种最常用的设计模式。所谓工厂模式就是将创建对象的过程单独封装。

工厂模式根据抽象程度的不同可以分为：
- 简单工厂模式（Simple Factory）
- 工厂方法模式（Factory Method）
- 抽象工厂模式（Abstract Factory）

这里我们要先理解什么是抽象。

> 抽象：将复杂事物的一个或多个共有特征抽取出来的思维过程。


### 简单工厂模式
简单工厂模式也叫静态工厂模式，用一个工厂对象创建同一类对象类的实例。

假设我们要开发一个公司岗位及其工作内容的录入信息，不同岗位的工作内容不一致。

代码如下：

```js
function Factory(career) {
    function User(career, work) {
        this.career = career 
        this.work = work
    }
    let work
    switch(career) {
        case 'coder':
            work =  ['写代码', '修Bug'] 
            return new User(career, work)
            break
        case 'hr':
            work = ['招聘', '员工信息管理']
            return new User(career, work)
            break
        case 'driver':
            work = ['开车']
            return new User(career, work)
            break
        case 'boss':
            work = ['喝茶', '开会', '审批文件']
            return new User(career, work)
            break
    }
}
let coder = new Factory('coder')
console.log(coder)
let boss = new Factory('boss')
console.log(boss)
```
`Factory`就是一个简单工厂。当我们调用工厂函数时，只需要传递name、age、career就可以获取到包含用户工作内容的实例对象。


简单工厂的优点就是我们只要传递正确的参数，就能获得所需的对象，而不需要关心其创建的具体细节。

应用场景也非常容易识别：有构造函数的地方，就应该想到简单工厂；在写了大量构造函数、调用了大量的new、自觉非常不爽的情况下，就应该思考是不是可以掏出工厂模式重构代码。

但是也不是所有情况都能简单工厂。比如：在函数内包含了所有对象的创建逻辑和判断逻辑代码，每增加新的构造函数还需要修改判断逻辑代码。如果我们的岗位不止上面的四个，而是1000个甚至更多，那么这个函数就会变得非常庞大，使得代码难以维护。所以简单工厂模式只能作用于创建的对象比较少，对象的创建逻辑不复杂时使用。

### 工厂方法模式
工厂方法模式是将创建对象的工作推到子类中进行，这样核心类就变成了抽象类。

也就是相当于工厂总部不生产产品了，交给下辖分工厂进行生产；但是进入工厂之前，需要有个判断来验证你要生产的东西是否是属于我们工厂所生产范围，如果是，就丢给下辖工厂来进行生产，如果不行，那么要么新建工厂生产要么就生产不了。


我们可以将工厂方法看作是一个实例化对象的工厂类。

我们对上面简单工厂模式的代码进行改造，刚才提到将工厂方法看作一个实例化对象的工厂，它只做实例化对象这一件事情。
```js
// 工厂方法
function Factory(career){
    if(this instanceof Factory){
        var a = new this[career]();
        return a;
    }else{
        return new Factory(career);
    }
}
// 工厂方法函数的原型中设置所有对象的构造函数
Factory.prototype={
    'coder': function(){
        this.careerName = '程序员'
        this.work = ['写代码', '修Bug'] 
    },
    'hr': function(){
        this.careerName = 'HR'
        this.work = ['招聘', '员工信息管理']
    },
    'driver': function () {
        this.careerName = '司机'
        this.work = ['开车']
    },
    'boss': function(){
        this.careerName = '老板'
        this.work = ['喝茶', '开会', '审批文件']
    }
}
let coder = new Factory('coder')
console.log(coder)
let hr = new Factory('hr')
console.log(hr)
```

使用工厂方法改造之后，如果我们需要添加新的岗位信息，只要在Factory.prototype中添加。

工厂方法关键核心代码是工厂里面的判断this是否属于工厂，也就是做了分支判断，这个工厂只做我能做的产品，如果你的产品我目前做不了，请找其他工厂代加工。

### 抽象工厂模式
上面介绍了简单工厂模式和工厂方法模式都是直接生成实例，但是抽象工厂模式不同，抽象工厂模式并不直接生成实例， 而是用于对产品类簇的创建。

通俗点来讲就是：简单工厂和工厂方法模式的工作是生产产品，那么抽象工厂模式的工作就是生产工厂的。

我们还是来看上面的例子：例子中有coder、hr、boss、driver四种岗位，其中coder可能使用不同的开发语言进行开发，比如JavaScript、Java等等。那么这两种语言就是对应的类簇。

在抽象工厂中，类簇一般用父类定义，并在父类中定义一些抽象方法，再通过抽象工厂让子类继承父类。所以，抽象工厂其实是实现子类继承父类的方法。

> 抽象方法：指声明但不能使用的方法。在Javascript中，abstract是保留字，我们一般通过在类的方法中抛出错误来模拟抽象类。

例如：
```js
let JavaCoder = function (){}
JavaCoder.prototype = {
  getCareerName: function(){
    return new Error('抽象方法不能调用')
  }
}
```

上面代码中的getCareerName就是抽象方法，我们虽然定义了它，但没有去实现。如果子类继承了父类，但没有去重写getCareerName，那么子类的实例化对象会调用父类的getCareerName方法并抛出错误提示。

下面我们先来实现岗位管理的抽象工厂方法：
```js
let CareerAbstractFactory = function(subType, superType) {
  // 判断抽象工厂中是否有该抽象类
  if (typeof CareerAbstractFactory[superType] === 'function') {
    // 缓存类
    function F() {}
    // 继承父类属性和方法
    F.prototype = new CareerAbstractFactory[superType]()
    // 将子类的constructor指向父类
    subType.constructor = subType;
    // 子类原型继承父类
    subType.prototype = new F()
  } else {
    throw new Error('抽象类不存在')
  }
}

// JavaScript开发者抽象类
CareerAbstractFactory.JavaScriptCoder = function (){
  this.language = 'javascript'
}
CareerAbstractFactory.JavaScriptCoder.prototype = {
  getCareerName: function(){
    return new Error('抽象方法不能调用')
  }
}

// Java开发者抽象类
CareerAbstractFactory.JavaCoder = function(){
  this.language = 'java'
}
CareerAbstractFactory.JavaCoder.prototype = {
  getCareerName: function(){
    return new Error('抽象方法不能调用')
  }
}
```
上面代码中CareerAbstractFactory就是一个抽象工厂方法，该方法在参数中传递子类和父类，在方法体内部实现了子类对父类的继承。对抽象工厂方法添加抽象类的方法我们是通过点语法进行添加的。

接下来我们来定义coder的子类：
```js
// JavaScriptCoder的子类
function CoderOfJavaScript (careerName) {
  this.careerName = careerName
  this.work = ['写代码', '修Bug']
}
// 抽象工厂实现JavaScriptCoder类的继承
CareerAbstractFactory(CoderOfJavaScript, 'JavaScriptCoder')
// 重写抽象方法
CoderOfJavaScript.prototype.getLanguage = function (){
  return this.careerName;
}


function CoderOfJava (careerName) {
  this.careerName = careerName
  this.work = ['写代码', '修Bug']
}
// 抽象工厂实现JavaScriptCoder类的继承
CareerAbstractFactory(CoderOfJava, 'JavaCoder')
// 重写抽象方法
CoderOfJava.prototype.getLanguage = function (){
  return this.careerName;
}
```
上面我们分别定义了CoderOfJavaScript、CoderOfJava两种类。这两个类作为子类通过抽象工厂方法实现继承。特别需要注意的是，调用抽象工厂方法后不要忘记重写抽象方法，否则在子类的实例中调用抽象方法会报错。

接下来我们进行实例化，检测抽象工厂方法是否实现了类簇的管理。

```js
let javaCode1 = new CoderOfJava('Java后端开发')
console.log(javaCode1.getCareerName(), javaCode1.language)
let javaCode2 = new CoderOfJava('Java大数据开发')
console.log(javaCode2.getCareerName(), javaCode2.language)

let javascriptCoder1 = new CoderOfJavaScript('前端开发')
console.log(javascriptCoder1.getCareerName(), javascriptCoder1.language);
let nodejsCoder = new CoderOfJavaScript('node全栈开发')
console.log(nodejsCoder.getCareerName(), nodejsCoder.language)
```
![](https://image.chenhanpeng.com/static/blog-images/blogImages/2021/spring/20210113125948.png)
从结果来看CareerAbstractFactory这个抽象工厂很好地实现了它的作用，将不同岗位按照开发语言这一类簇进行了分类。

抽象工厂的作用：它不直接创建实例，而是通过类的继承进行类簇的管理。

抽象工厂模式一般用于严格要求以面向对象思想进行开发的超大型项目中，我们一般常规的开发的话一般就是简单工厂和工厂方法模式会用的比较多一些


> 上面的代码我们是用ES5写的，ES6中提供了class语法，虽然class本质上是一颗语法糖，并也没有改变JavaScript是使用原型继承的语言，但是确实让对象的创建和继承的过程变得更加的清晰和易读。有兴趣的可以自己尝试用ES6的新语法重写上面三个例子。

## 总结
工厂模式是属于创建型的设计模式。简单工厂模式又叫静态工厂方法，用来创建某一种产品对象的实例，用来创建单一对象；工厂方法模式是将创建实例推迟到子类中进行；抽象工厂模式是对类的工厂抽象用来创建产品类簇，不负责创建某一类产品的实例。

**工厂模式的优点：**
- 创建对象过程可能很复杂，但我们只需要关心创建结果
- 构造函数和创建者分离，符合“开闭原则”
- 一个调用者想创建一个对象，只要知道其名称就可以了。
- 扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以。

**工厂模式的缺点：**
- 添加新产品时，需要编写新的具体产品类,一定程度上增加了系统的复杂度
- 考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度

**适用场景：**
- 如果你不想让某个子系统与较大的那个对象之间形成强耦合，而是想运行时从许多子系统中进行挑选的话，那么工厂模式是一个理想的选择
- 将new操作简单封装，遇到new的时候就应该考虑是否用工厂模式；
- 需要依赖具体环境创建不同实例，这些实例都有相同的行为,这时候我们可以使用工厂模式，简化实现的过程，同时也可以减少每种对象所需的代码量，有利于消除对象间的耦合，提供更大的灵活性

本文中示例代码：[工厂模式示例代码](https://github.com/Hanpeng-Chen/html-js-demo-code/tree/main/design-pattern/factory-method-pattern)

## 参考文章
- [JavaScript 设计模式核⼼原理与应⽤实践](https://juejin.cn/book/6844733790204461070)

- [JavaScript设计模式es6（23种)](https://juejin.cn/post/6844904032826294286#heading-21)