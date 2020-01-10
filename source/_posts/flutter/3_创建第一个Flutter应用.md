---
title: Flutter | 创建第一个Flutter应用
date: 2020-01-03 21:53:21
urlname: create-first-flutter-app
tags:
  - 第一个Fultter应用
categories:
  - 前端
  - Flutter
---
用Android Studio和VS Code创建的Flutter应用模板默认是一个简单的计数器示例。今天我们通过创建的计数器示例来了解Flutter应用程序的结构。

# 1、创建Flutter应用模板
通过命令行`flutter create <output directory>`创建一个名为 first_flutter_application的Flutter工程。

运行创建好的工程，计数器例子效果如下图：

{% qnimg flutter/3-1.png %}

在这个例子中，点击右下角“+”悬浮按钮，屏幕中间的数字则会+1。

这个示例的主要Dart代码在 `lib/main.dart` 文件中，源码如下：
```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);

  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.display1,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}
```
## 代码说明

1、导入包
```dart
import 'package:flutter/material.dart';
```
上面这行代码作用是导入`Material UI`组件库。Material是一种标准的移动端和web端的视觉设计语言，Flutter默认提供了一套丰富的Material风格的UI组件。

2、应用入口
```dart
void main() => runApp(MyApp());
```
- 和java类似，main函数在Flutter中也是应用程序的入口。main函数中调用了 runApp 方法，它的功能是启动Flutter应用。runApp 它接受一个 Widget 参数，在计数器这个示例中，它是一个 MyApp 对象，MyApp() 是Flutter应用的根组件。

- main函数使用了 ( => )符号，这是Dart中单行函数或方法的简写。

3、应用结构
```dart
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}
```

- MyApp类代表Flutter应用，它继承了 `StatelessWidget` 类，这也就意味着应用本身也是一个widget。

- 在Flutter中，大多数东西都是widget（后同“组件”或“部件”），包括对齐(alignment)、填充(padding)和布局(layout)等，它们都是以widget的形式提供。

- Flutter在构建页面时，会调用组件的build方法，widget的主要工作是提供一个build()方法来描述如何构建UI界面（通常是通过组合、拼装其它基础widget）。

- `MaterialApp` 是Material库中提供的Flutter APP框架，通过它可以设置应用的名称、主题、语言、首页及路由列表等。MaterialApp也是一个widget。

- `home` 为Flutter应用的首页，它也是一个widget。

# 2、首页
## StatefulWidget类
```dart
class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);

  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  ...
}
```
MyHomePage 是Flutter应用的首页，它继承自 StatefulWidget 类，表示它是一个有状态的组件（Stateful widget）。关于StatefulWidget 和 StatelessWidget 在后面我们会详细介绍，这里我们先简单两个这两者有下面两点不同：
- `StatefulWidget` 可以拥有状态，这些状态在widget生命周期中是可以改变的，而StatelessWidget是不可以变的。
- `StatefulWidget` 至少由两个类组成：StatefulWidget类和 State类。 StatefulWidget类本身是不变的，但是State类中持有的状态在widget生命周期中可能会发生变化。

在上面的代码中，_MyHomePageState类是MyHomePage类对应的状态类。

## State类
接下来，我们看下_MyHomePageState中都包含哪些东西：

1、该组件的状态。由于我们只需要维护一个点击次数计数器，所以定义一个 _counter 状态：
```
int _counter = 0;
```

`_counter` 为保存屏幕右下角悬浮按钮点击次数的状态。

2、设置状态的自增函数
```dart
void _incrementCounter() {
  setState(() {
    _counter++;
  })
}
```
当按钮点击时，会调用此函数，该函数的作用是先自增_counter，然后调用setState 方法。setState方法的作用是通知Flutter框架，有状态发生了改变，Flutter框架收到通知后，会执行build方法来根据新的状态重新构建界面， Flutter 对此方法做了优化，使重新执行变的很快，所以你可以重新构建任何需要更新的东西，而无需分别去修改各个widget。

3、构建UI界面

构建UI界面的逻辑在build方法中，当MyHomePage第一次创建时，_MyHomePageState类会被创建，当初始化完成后，Flutter框架会调用Widget的build方法来构建widget树，最终将widget树渲染到设备屏幕上。所以，我们看看_MyHomePageState的build方法中都干了什么事：
```dart
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text(widget.title),
      ),
      body: new Center(
        child: new Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            new Text(
              'You have pushed the button this many times:',
            ),
            new Text(
              '$_counter',
              style: Theme.of(context).textTheme.display1,
            ),
          ],
        ),
      ),
      floatingActionButton: new FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: new Icon(Icons.add),
      ),
    );
  }
```
- `Scaffold` 是 `Material` 库中提供的页面脚手架，它提供了默认的导航栏、标题和包含主屏幕widget树（后同“组件树”或“部件树”）的body属性，组件树可以很复杂。本书后面示例中，路由默认都是通过Scaffold创建。

- body的组件树中包含了一个Center 组件，`Center` 可以将其子组件树对齐到屏幕中心。此例中， Center 子组件是一个Column 组件，Column的作用是将其所有子组件沿屏幕垂直方向依次排列； 此例中Column子组件是两个 Text，第一个Text 显示固定文本 “You have pushed the button this many times:”，第二个Text 显示_counter状态的数值。

- floatingActionButton是页面右下角的带“+”的悬浮按钮，它的onPressed属性接受一个回调函数，代表它被点击后的处理器，本例中直接将_incrementCounter方法作为其处理函数。


现在，我们将整个计数器执行流程串起来：当右下角的floatingActionButton按钮被点击之后，会调用_incrementCounter方法。在_incrementCounter方法中，首先会自增_counter计数器（状态），然后setState会通知Flutter框架状态发生变化，接着，Flutter框架会调用build方法以新的状态重新构建UI，最终显示在设备屏幕上。

# 为什么将build方法放在State中，而不放在StatefulWidget中？
从示例中我们发现：和MyApp 类不同， MyHomePage类中并没有build方法，取而代之的是，build方法被挪到了_MyHomePageState方法中，至于为什么这么做？这主要是为了提高开发的灵活性。如果将build()方法放在StatefulWidget中则会有两个问题：

- **状态访问不便**

试想一下，如果我们的StatefulWidget有很多状态，而每次状态改变都要调用build方法，由于状态是保存在State中的，如果build方法在StatefulWidget中，那么build方法和状态分别在两个类中，那么构建时读取状态将会很不方便！试想一下，如果真的将build方法放在StatefulWidget中的话，由于构建用户界面过程需要依赖State，所以build方法将必须加一个State参数，大概是下面这样：
```dart
  Widget build(BuildContext context, State state){
      //state.counter
      ...
  }
```
这样的话就只能将State的所有状态声明为公开的状态，这样才能在State类外部访问状态！但是，将状态设置为公开后，状态将不再具有私密性，这就会导致对状态的修改将会变的不可控。但如果将build()方法放在State中的话，构建过程不仅可以直接访问状态，而且也无需公开私有状态，这会非常方便。


- **继承StatefulWidget不便**
例如，Flutter中有一个动画widget的基类AnimatedWidget，它继承自StatefulWidget类。AnimatedWidget中引入了一个抽象方法build(BuildContext context)，继承自AnimatedWidget的动画widget都要实现这个build方法。现在设想一下，如果StatefulWidget 类中已经有了一个build方法，正如上面所述，此时build方法需要接收一个state对象，这就意味着AnimatedWidget必须将自己的State对象(记为_animatedWidgetState)提供给其子类，因为子类需要在其build方法中调用父类的build方法，代码可能如下：
```dart
class MyAnimationWidget extends AnimatedWidget{
    @override
    Widget build(BuildContext context, State state){
      //由于子类要用到AnimatedWidget的状态对象_animatedWidgetState，
      //所以AnimatedWidget必须通过某种方式将其状态对象_animatedWidgetState
      //暴露给其子类   
      super.build(context, _animatedWidgetState)
    }
}
```

这样很显然是不合理的，因为

1、AnimatedWidget的状态对象是AnimatedWidget内部实现细节，不应该暴露给外部。

2、如果要将父类状态暴露给子类，那么必须得有一种传递机制，而做这一套传递机制是无意义的，因为父子类之间状态的传递和子类本身逻辑是无关的。

综上所述，可以发现，对于StatefulWidget，将build方法放在State中，可以给开发带来很大的灵活性。