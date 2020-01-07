---
title: 了解Flutter
urlname: start-learn-flutter
date: 2019-12-23 23:08:34
tags:
  - Flutter
categories:
  - 前端
  - Flutter
---
# 1、Flutter
Flutter是Google推出并开源的移动应用开发框架，主打跨平台、高保真、高性能。开发者可以通过Dart语言开发APP，一套代码可以同时运行在ios、android、web。Flutter提供了丰富的组件、接口，开发者可以很快为Flutter添加native扩展。同时Flutter还使用Native引擎渲染视图，这为用户提供良好的体验。

## 跨平台自绘引擎
Flutter和其他大多数构建移动应用程序的框架不同，Flutter既不使用WebView，也不使用操作系统的原生控件。相反，Flutter使用自己的高性能渲染引擎来绘制widget。这样不仅可以保证Android和iOS上UI的一致性，而且可以避免对原生控件依赖而带来的限制及高昂的维护成本。

FLutter使用Skia作为2D渲染引擎，Skia是Google的一个2D图形处理函数库，包含字型、坐标转换，以及点阵图都有高效能且简洁的表现，Skia是跨平台的，并提供了非常友好的API，目前Google Chrome浏览器和Android均采用Skia作为其绘图引擎。

目前Flutter默认支持iOS、Android、Fuchsia三个移动平台。但Flutter也可支持web开发和PC开发。这里我们主要学习其基于iOS和Android平台的开发。

## 高性能
Flutter高性能主要靠两点来保证，首先，Flutter APP采用Dart语言开发。Dart在 JIT（即时编译）模式下，速度与 JavaScript基本持平。但是 Dart支持 AOT，当以 AOT模式运行时，JavaScript便远远追不上了。速度的提升对高帧率下的视图数据计算很有帮助。其次，Flutter使用自己的渲染引擎来绘制UI，布局数据等由Dart语言直接控制，所以在布局过程中不需要像RN那样要在JavaScript和Native之间通信，这在一些滑动和拖动的场景下具有明显优势，因为在滑动和拖动过程往往都会引起布局发生变化，所以JavaScript需要和Native之间不停的同步布局信息，这和在浏览器中要JavaScript频繁操作DOM所带来的问题是相同的，都会带来比较可观的性能开销。

## 采用Dart语言开发
当前，程序主要有两种运行方式：静态编译与动态编译。静态编译的程序在执行前全部被翻译成机器码，通常将这种类型称为AOT（Ahead of time）即“提前编译”；而解释执行的则是一句一句边翻译边运行，通常将这种类型称为JIT（just-in-time）即“即时编译”。

AOT程序的典型代表是用C/C++开发的应用，它们必须在执行前编译成机器码，而JIT的代表非常多，比如JavaScript、Python等等，事实上，所有脚本语言都支持JIT模式。但需要注意的是JIT和AOT指的是程序运行方式，和编程语言并非强关联的，有些语言既可以以JIT方式运行也可以以AOT方式运行，如Java、Python，它们可以在第一次执行时编译成中间字节码、然后在之后执行时可以直接执行字节码，也许有人会说，中间字节码并非机器码，在程序执行时仍然需要动态将字节码转为机器码，是的，这没有错，不过通常我们区分是否为AOT的标准就是看代码在执行之前是否需要编译，只要需要编译，无论其编译产物是字节码还是机器码，都属于AOT。

Dart和JavaScript对比的优势：
- 开发效率高
  Dart运行时和编译器支持Flutter的两个关键特性的组合：
  基于JIT的快速开发周期：Flutter在开发阶段采用JIT模式，避免了每次改动都需要进行编译，极大节省开发时间。
  基于AOT的发布包：Flutter在发布时可以通过AOT生成高效的ARM代码以保证应用性能。而JavaScript则不具有这个能力。

- 高性能
  Flutter旨在提供流畅、高保真的的UI体验。为了实现这一点，Flutter中需要能够在每个动画帧中运行大量的代码。这意味着需要一种既能提供高性能的语言，而不会出现会丢帧的周期性暂停，而Dart支持AOT，在这一点上可以做的比JavaScript更好。

- 快速内存分配

Flutter框架使用函数式流，这使得它在很大程度上依赖于底层的内存分配器。因此，拥有一个能够有效地处理琐碎任务的内存分配器将显得十分重要，在缺乏此功能的语言中，Flutter将无法有效地工作。当然Chrome V8的JavaScript引擎在内存分配上也已经做的很好，事实上Dart开发团队的很多成员都是来自Chrome团队的，所以在内存分配上Dart并不能作为超越JavaScript的优势，而对于Flutter来说，它需要这样的特性，而Dart也正好满足而已。

- 类型安全

由于Dart是类型安全的语言，支持静态类型检测，所以可以在编译前发现一些类型的错误，并排除潜在问题，这一点对于前端开发者来说可能会更具有吸引力。与之不同的，JavaScript是一个弱类型语言，也因此前端社区出现了很多给JavaScript代码添加静态类型检测的扩展语言和工具，如：微软的TypeScript以及Facebook的Flow。相比之下，Dart本身就支持静态类型，这是它的一个重要优势。

# 2、Flutter框架结构
我们先看一下Flutter官方提供的Flutter框架图，如下图所示：
<!-- ![](/images/articles/flutter/1-1.png) -->
{% qnimg flutter/1-1.png %}

## Flutter Framework
这是一个纯Dart实现的SDK，它实现了一套基础库。
- 底下两层（Foundation和Animation、Painting、Gestures）在Google的一些视频中被合并为一个Dart UI层，对应的是Flutter中的dart:ui包，它是Flutter引擎暴露的底层UI库，提供动画、手势及绘制能力。
- Rendering层：这一层是一个抽象的布局层，它依赖于dart UI层，Rendering层会构建一个UI树，当UI树有变化时，会计算出有变化的部分，然后更新UI树，最终将UI树绘制到屏幕上，这个过程类似于React中的虚拟DOM。Rendering层可以说是Flutter UI 框架最核心的部分，它除了确定每个UI元素的位置、大小之外还要进行坐标变换、绘制（调用底层dart:ui）。
- Widgets层是Flutter提供的一套基础组件库，在基础组件库之上，Flutter还提供了Material和Cupertino两种视觉风格的组件库。我们Flutter开发大多数场景就是和这两层打交道。

## Flutter Engine
这是一个纯C++实现的SDK，其中包括了Skia引擎、Dart运行时、文字排版引擎等等。在代码调用dart:ui库时，调用最终会走到Engine层，然后实现真正的绘制逻辑。


关于Flutter的安装请参考Flutter中文网的安装教程：
https://flutterchina.club/get-started/install/