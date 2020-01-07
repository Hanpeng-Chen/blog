---
title: TensorFlow基础概念
urlname: Basic-concepts-of-TensorFlow
tags:
  - TensorFlow
categories:
  - 机器学习
date: 2019-06-11 23:25:14
---

TensorFlow是一个采用数据流图（data flow graphs），用于数值计算的开源软件库。

TensorFlow 是Google第二代大规模分布式深度学习框架。

- 灵活通用的深度学习库
- 端云结合的人工智能引擎
- 高性能的基础平台软件
- 跨平台的机器学习系统

**应用场景：**

行人检测、人脸检测、行为识别、身份证自动输入与人脸图像比较、OCR+自动化审核


## TensorFlow 数据流图介绍

TensorFlow数据流图是一种声明式编程范式

声明式编程与命令式编程的多角度对比
<!-- ![Alt](/images/articles/2019/TensorFlow/tensorflow-base-liutu-1.png) -->
{% qnimg TensorFlow/tensorflow-base-liutu-1.png %}

数据流图由有向边和节点组成
<!-- ![Alt](/images/articles/2019/TensorFlow/tensorflow-base-liutu-2.png) -->
{% qnimg TensorFlow/tensorflow-base-liutu-2.png %}

数据流图的优势：**快**
- 并行计算快
- 分布式计算快(CPUs,GPUs,TPUs)
- 预编译优化(XLA)
- 可移植性好(Language-independent representation)


## 张量(Tensor)

在数学里，张量是一种几何实体，广义上表示任意形式的“数据”。张量可以理解为0阶（rank）标量、1阶向量和2阶矩阵在高维度空间上的推广，张量的阶描述它表示数据的最大维度。

在TensorFlow中，张量表示某种相同的数据类型的多维数组
因此，张量有两个重要的属性：

1、**数据类型**，如浮点型、整型、字符串

2、**数组形状**： 各个维度的大小

TensorFlow张量是什么可以总结为一下几点：
- 张量是用来表示多维数据的
- 张量是执行操作时的输入或输出数据
- 用户通过执行操作来创建或计算张量
- 张量的形状不一定在编译时确定，可以在运行时通过形状推断计算得到


几类比较特别的张量，由以下操作产生：
```
tf.constant     // 常量
tf.placeholder   // 占位符
tf.Variable      // 变量
```

## 变量(Variable)

变量Variable是一种特殊的张量，主要作用是维护特定节点的状态，如深度学习或机器学习的模型参数。

tf.Variable方法是操作，返回值是变量（特殊张量）

通过tf.Variable方法创建的变量，与张量一样，可以作为操作的输入和输出。

不同之处：
- 张量的生命周期通常随依赖的计算完成而结束，内存也随即释放
- 变量则常驻内存，在每一步训练时不断更新值，以实现模型参数的更新。

### TensorFlow变量使用流程
<!-- ![Alt](/images/articles/2019/TensorFlow/tensorflow-base-variable-1.png) -->
{% qnimg TensorFlow/tensorflow-base-variable-1.png %}


## 操作(Operation)

TensorFlow用数据流图表示算法模型。数据流图由节点和有向边组成，每个节点均对应一个具体的操作。因此，操作是模型功能的**实际载体**。

数据流图中的节点按照功能不同可以分为3种：

* **存储节点**：有状态的变量操作，通常用来存储模型参数
* **计算节点**：无状态的计算或控制操作，主要负责算法逻辑表达或流程控制。
* **数据节点**：数据的占位符操作，用于描述图外输入数据的属性。

**操作的输入和输出是张量或操作（函数式编程）**

### TensorFlow典型计算和控制操作

<!-- ![Alt](/images/articles/2019/TensorFlow/tensorflow-base-operation-1.png) -->
{% qnimg TensorFlow/tensorflow-base-operation-1.png %}

### TensorFlow占位符操作

tensorflow 使用占位符操作表示图外输入的数据，如训练数据和测试数据。

TensorFlow数据流图描述了算法模型的计算拓扑，其中的各个操作(节点)都是抽象的函数映射或数学表达式。

换句话说，数据流图本身是一个具有计算拓扑和内部结构的“壳”。在用户向数据流图填充数据前，图中并没有真正执行任何计算。


## 会话(Session)

会话提供了估算张量和执行操作的运行环境，它是发放计算任务的客户端，所有计算任务都由它连接的执行引擎完成。一个会话的典型使用流程分为以下3步：
```
# 1、创建会话
# target:会话连接的执行引擎  graph:会话加载的数据流图  config:会话启动时的配置项
sess = tf.Session(target=..., graph=..., config=...)
# 2、估算张量或执行操作
sess.run(...)
# 3、关闭会话
sess.close()
```

### 会话执行
获取张量值的另外两种方法：估算张量(Tensor.eval)与执行操作(Operation.run)

```
import tensorflow as tf
# 创建数据流图：y=w * x + b, 其中w和b为存储节点，x为数据节点
x = tf.placeholder(tf.float32)
w = tf.Variable(1.0)
b = tf.Variable(1.0)
y = w * x + b
with tf.Session() as sess:
    tf.global_variables_initializer().run() # Operation.run
    fetch = y.eval(feed_dict={x: 3.0}) # Tensor.eval
    print(fetch)   # fetch = 1.0 * 3.0 + 1.0
```

### 会话执行原理

当我们调用sess.run(train_op)语句执行训练操作时：

- 首先，程序内部提取操作依赖的所有前置操作。这些操作的节点共同组成一幅子图。
- 然后，程序会将子图中的计算节点、存储节点和数据节点按照各自的执行设备分类，相同设备上的节点组成了一幅局部图
- 最后，每个设备上的局部图在实际执行时，根据节点间的依赖关系将各个节点有序地加载到设备上执行。



## 优化器(Optimizer)

优化器是实现优化算法的载体。

一次典型的迭代优化应该分为以下3个步骤：

1、**计算梯度**：调用compute_gradients方法;

2、**处理梯度**：用户按照自己需求处理梯度值，如梯度裁剪和梯度加权等。

3、**应用梯度**：调用apply_gradients方法，将处理后的梯度值应用到模型参数。

TensorFlow内置优化器：
<!-- ![Alt](/images/articles/2019/TensorFlow/tensorflow-base-optimizer.png) -->
{% qnimg TensorFlow/tensorflow-base-optimizer.png %}
