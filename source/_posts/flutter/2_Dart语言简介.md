---
title: Flutter | 2-Dart语言简介
urlname: dart-introduction
tags:
  - Dart语法
categories:
  - 前端
  - Flutter
abbrlink: 16952
date: 2019-12-23 23:30:34
---
Dart在静态语法方面和Java非常相似，如类型定义、函数声明、泛型等，而在动态特性方面又和JavaScript很想，如函数式特性、异步支持等。除了融合Java和JavaScript语言的长处外，Dart也具有一些其他具有表现力的语法，如可选命名参数、..(联级运算符)和 ?. （条件成员访问运算符）以及 ?? （判空赋值运算符）。

Flutter是采用Dart语言进行开发的，所以我们来学习一下Dart在Flutter开发中常用的语法特性。

下面例子可以在DartPad上面进行运行校验：https://dartpad.cn/

# 变量声明
## 1. var
类似于JavaScript中var，它可以接收任何类型的变量，但和JavaScript最大的不同在于：Dart中var变量一单赋值，类型便会确定，不能在改变其类型。
```dart
void main() {
  var t;
  t = "hello world!";
  t = 1;
}
```
上面的代码在JavaScript是没有问题的，前端开发者需要注意一下，之所以有此差异是因为Dart本身是一个强类型语言，任何变量都是有确定类型的，在Dart中，当用var声明一个变量后，Dart在编译时会根据第一次赋值数据的类型来推断其类型，编译结束后其类型就已经被确定，而JavaScript是纯粹的弱类型脚本语言，var只是变量的声明方式而已。

## 2. dynamic和Object
Object是Dart所有对象的根基类，也就是说所有类型都是Object子类（包括Function和Null），所以任何类型的数据都可以赋值给Object声明的对象。dynamic和var一样都是关键词，声明的变量可以赋值任何对象。而dynamic和Object相同之处在于：他们声明的变量可以在后期改变赋值类型。
```dart
void main() {
  dynamic user;
  Object system;
  user = 'zhangsan';
  system = 'dart';
  print(user);
  print(system);
  user = {
    'name': 'zhangsan',
    'age': 20
  };
  system = 1;
  print(user);
  print(system);
}
```
dynamic与Object不同的是,dynamic声明的对象编译器会提供所有可能的组合, 而Object声明的对象只能使用Object的属性与方法, 否则编译器会报错。如:
```dart
void main() {
  dynamic user;
  Object system;
  user = 'zhangsan';
  system = 'dart';
  print(user.length);
  print(system.length);
}
```
变量user不会报错, 变量system编译器会报错

dynamic的这个特性与Objective-C中的id作用很像. dynamic的这个特点使得我们在使用它时需要格外注意,这很容易引入一个运行时错误.

## 3. final 和 const
如果你从未打算更改一个变量，那么使用final或const，不是var，也不是一个类型。一个final变量只能被设置一次，两者区别在于：const变量是一个编译时常量，final变量在第一次使用时被初始化。被final或者const修饰的变量，变量类型可以省略。
```dart
void main() {
  final str = 'hello world!';
  const str1 = 'hello world';
}
```

# 函数
Dart是一种真正的面向对象的语言，所以即使是函数也是对象，并且有一个类型Function。这意味着函数可以赋值给变量或作为参数传递给其他函数，这是函数式编程的典型特征。

## 1、函数声明
```dart
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
```
Dart函数声明如果没有显式声明返回值类型是会默认当做dynamic处理，注意，函数返回值没有类型推断：
```dart
typedef bool CALLBACK();

//不指定返回类型，此时默认为dynamic，不是bool
isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

void test(CALLBACK cb){
   print(cb()); 
}
//报错，isNoble不是bool类型
test(isNoble);
```

## 2、对于只包含一个表达式的函数，可以使用简写语法
```dart
bool isNoble (int atomicNumber) => _nobleGases[atomicNumber] != null;
```

## 3、函数作为变量
```dart
var say = (str) {
  print(str)
};
say('hello world!');
```

## 4、函数作为参数传递
```dart
void execute(var callback) {
  callback();
}
execute(() => print('hello world'));
```

## 5、可选的位置参数
包装一组函数参数，用[]标记为可选的位置参数，并放在参数列表的最后面：
```dart
String say(String from, String msg, [String device]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  return result;
}
```
下面是一个不带可选参数调用这个函数的例子：
```
say('iphoneX', 'system error');
```
下面是用第三个参数调用这个函数的例子：
```
say('iphoneX', 'system error', 'signal');
```

## 6、可选的命名参数
定义函数时，使用{param1, param2, ...}，放在参数列表的最后面，用于指定命名参数。例如：
```
//设置[bold]和[hidden]标志
void enableFlags({bool bold, bool hidden}) {
    // ... 
}
```
调用函数时，可以使用指定命名参数。例如：paramName: value
```
enableFlags(bold: true, hidden: false);
```
可选命名参数在Flutter中使用非常多。

> 注意，不能同时使用可选的位置参数和可选的命名参数

# 异步支持
Dart类库有非常多返回Future或者Stream对象的函数。这些函数被称为**异步函数**：它们只会在设置好一些耗时操作之后返回，比如像IO操作。而不是等到这个操作完成。

`async`和 `await`关键词支持了异步编程，允许您写出和同步代码很像的异步代码。

## Future
`Future`与JavaScript中的`Promise`非常相似，表示一个异步操作的最终完成（或失败）及其结果值的表示。简单来说，它就是用于处理异步操作的，异步处理成功了就执行成功的操作，异步处理失败了就捕获错误或者停止后续操作。一个Future只会对应一个结果，要么成功，要么失败。

由于本身功能较多，这里我们只介绍其常用的API及特性。还有，请记住，Future 的所有API的返回值仍然是一个Future对象，所以可以很方便的进行链式调用。

### Future.then
下面我们利用Future.delayed 创建一个延时任务来模拟请求远程接口返回。
```dart
Future.delayed(new Duration(seconds: 3), (){
  return 'hello world';
}).then((data) {
  print(data);
});
```

### Future.catchError
如果异步任务发生错误，我们可以在catchError中捕获错误，我们将上面示例改为：
```dart
Future.delayed(new Duration(seconds: 3), (){
  throw AssertionError("Error");
}).then((data) {
  print(data);
}).catchError((e) {
  print(e);
});
```
在上面的示例中，我们在异步任务中抛出了一个异常，then的回调函数将不会被执行，取而代之的是 catchError回调函数将被调用；但是，并不是只有 catchError回调才能捕获错误，then方法还有一个可选参数onError，我们也可以它来捕获异常：
```
Future.delayed(new Duration(seconds: 3), (){
  throw AssertionError("Error");
}).then((data) {
  print(data);
}, onError: (e) {
  print(e);
});
```

### Future.whenComplete
有些时候，我们会遇到无论异步任务执行成功或失败都需要做一些事的场景，比如在网络请求前弹出加载对话框，在请求结束后关闭对话框。这种场景，有两种方法，第一种是分别在then或catch中关闭一下对话框，第二种就是使用Future的whenComplete回调，我们将上面示例改一下：
```
Future.delayed(new Duration(seconds: 3), (){
  throw AssertionError("Error");
}).then((data) {
  print(data);
}).catchError((e) {
  print(e);
}).whenComplete((){
  // 无论成功或失败都会走到这里
});
```

### Future.wait
有些时候，我们需要等待多个异步任务都执行结束后才进行一些操作，比如我们有一个界面，需要先分别从两个网络接口获取数据，获取成功后，我们需要将两个接口数据进行特定的处理后再显示到UI界面上，应该怎么做？答案是Future.wait，它接受一个Future数组参数，只有数组中所有Future都执行成功后，才会触发then的成功回调，只要有一个Future执行失败，就会触发错误回调。下面，我们通过模拟Future.delayed 来模拟两个数据获取的异步任务，等两个异步任务都执行成功时，将两个异步任务的结果拼接打印出来，代码如下：
```
Future.wait([
  // 2秒后返回结果  
  Future.delayed(new Duration(seconds: 2), () {
    return "hello";
  }),
  // 4秒后返回结果  
  Future.delayed(new Duration(seconds: 4), () {
    return " world";
  })
]).then((results){
  print(results[0]+results[1]);
}).catchError((e){
  print(e);
});
```
执行上面代码，4秒后你会在控制台中看到“hello world”。

## Async/await
如果代码中有大量异步逻辑，并且出现大量异步任务依赖其他异步任务的结果时，必然会出现`Future.then`回调中套回调情况。举个例子，比如有个场景是用户需要先登录，登录成功后获得用户Id，然后根据用户ID去请求个人信息，成功获取信息后将其缓存在本地文件系统中。代码示例如下：
```dart
//先分别定义各个异步任务
Future<String> login(String userName, String pwd){
    ...
    //用户登录
};
Future<String> getUserInfo(String id){
    ...
    //获取用户信息 
};
Future saveUserInfo(String userInfo){
    ...
    // 保存用户信息 
};
```
接下来执行整个流程：
```dart
login("alice","******").then((id){
 //登录成功后通过，id获取用户信息    
 getUserInfo(id).then((userInfo){
    //获取用户信息后保存 
    saveUserInfo(userInfo).then((){
       //保存用户信息，接下来执行其它操作
        ...
    });
  });
})
```
可以感受一下，如果业务逻辑中有大量异步依赖的情况，将会出现上面这种在回调里面套回调的情况，过多的嵌套会导致的代码可读性下降以及出错率提高，并且非常难维护，这个问题被形象的称为**回调地狱（Callback Hell）**。回调地狱问题在之前JavaScript中非常突出，也是JavaScript被吐槽最多的点，但随着ECMAScript6和ECMAScript7标准发布后，这个问题得到了非常好的解决，而解决回调地狱的两大神器正是ECMAScript6引入了`Promise`，以及ECMAScript7中引入的`async/await`。 而在Dart中几乎是完全平移了JavaScript中的这两者：Future相当于Promise，而async/await连名字都没改。接下来我们看看通过`Future`和`async/await`如何消除上面示例中的嵌套问题。

### 使用Future消除Callback Hell
```dart
login("alice","******").then((id){
      return getUserInfo(id);
}).then((userInfo){
    return saveUserInfo(userInfo);
}).then((e){
   //执行接下来的操作 
}).catchError((e){
  //错误处理  
  print(e);
});
```
正如上文所述， Future 的所有API的返回值仍然是一个Future对象，所以可以很方便的进行链式调用” ，如果在then中返回的是一个Future的话，该future会执行，执行结束后会触发后面的then回调，这样依次向下，就避免了层层嵌套。

### 使用async/await消除callback hell
通过Future回调中再返回Future的方式虽然能避免层层嵌套，但是还是有一层回调，有没有一种方式能够让我们可以像写同步代码那样来执行异步任务而不使用回调的方式？答案是肯定的，这就要使用async/await了，下面我们先直接看代码，然后再解释，代码如下：
```
task() async {
   try{
    String id = await login("alice","******");
    String userInfo = await getUserInfo(id);
    await saveUserInfo(userInfo);
    //执行接下来的操作   
   } catch(e){
    //错误处理   
    print(e);   
   }  
}
```
- `async`用来表示函数是异步的，定义的函数会返回一个Future对象，可以使用then方法添加回调函数。
- `await` 后面是一个Future，表示等待该异步任务完成，异步完成后才会往下走；await必须出现在 async 函数内部。

可以看到，我们通过async/await将一个异步流用同步的代码表示出来了。

> 其实，无论是在JavaScript还是Dart中，async/await都只是一个语法糖，编译器或解释器最终都会将其转化为一个Promise（Future）的调用链。

# Stream
Stream也是用于接收异步事件数据，和Future不同的是，它可以接收多个异步操作的结果（成功或失败）。也就是说，在执行异步任务时，可以通过多次触发成功或失败事件来传递结果数据或错误异常。 Stream 常用于会多次读取数据的异步任务场景，如网络内容下载、文件读写等。举个例子：
```
Stream.fromFutures([
  // 1秒后返回结果
  Future.delayed(new Duration(seconds: 1), () {
    return "hello 1";
  }),
  // 抛出一个异常
  Future.delayed(new Duration(seconds: 2),(){
    throw AssertionError("Error");
  }),
  // 3秒后返回结果
  Future.delayed(new Duration(seconds: 3), () {
    return "hello 3";
  })
]).listen((data){
   print(data);
}, onError: (e){
   print(e.message);
},onDone: (){

});
```
执行上面代码依次输出：
```
hello 1
Error
hello 3
```