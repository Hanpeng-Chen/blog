---
title: Python从小白到攻城狮(1)——python环境搭建
urlname: python-1-env
date: 2019-08-06 20:31:03
tags:
  - python教程
  - python环境搭建
categories:
  - Python
---

# Python介绍
---
Python是Guido van Rossum在1989年圣诞节期间，为了打发无聊的圣诞节而编写的一个编程语言，1991年发布第一版。

Python 是一个高层次的结合了解释性、编译性、互动性和面向对象的脚本语言。

Python 的设计具有很强的可读性，相比其他语言经常使用英文关键字，其他语言的一些标点符号，它具有比其他语言更有特色语法结构。

- Python 是一种解释型语言： 这意味着开发过程中没有了编译这个环节。类似于PHP和Perl语言。

- Python 是交互式语言： 这意味着，您可以在一个 Python 提示符 >>> 后直接执行代码。

- Python 是面向对象语言: 这意味着Python支持面向对象的风格或代码封装在对象的编程技术。

- Python 是初学者的语言：Python 对初级程序员而言，是一种伟大的语言，它支持广泛的应用程序开发，从简单的文字处理到 WWW 浏览器再到游戏。

## Python的应用领域

目前Python在Web应用开发、云基础设施、DevOps、网络爬虫开发、数据分析挖掘、机器学习等领域都有着广泛的应用，因此也产生了Web后端开发、数据接口开发、自动化运维、自动化测试、科学计算和可视化、数据分析、量化交易、机器人开发、图像识别和处理等一系列的职位。


# Python环境搭建
---
对于刚开始学习Python的新手，建议安装Anaconda。win 下安装包的时候用 anaconda 比 pip 安装要好一些，pip 有时候会因为一些依赖导致安装失败，这时候anaconda就体现出它对新手的友好。

搜索Anaconda进入官网或点击下方官网链接进入

https://www.anaconda.com/distribution/

在页面我们可以看到windows、macOS、linux对应的安装包。

<!-- ![](/images/articles/2019/python_series/install_python_1.png) -->
{% qnimg python_series/install_python_1.png %}

如果是初学者，建议下载Python3.X版本，而不是Python2.X。因为python的2和3版本的语法是有差异的，Python2.X将在2020年4月后不再进行任何维护。

下载安装包，双击安装。

划重点：安装过程中最好将下图所示的`添加到环境变量`的选项勾上。

<!-- ![](/images/articles/2019/python_series/install_python_2.png) -->
{% qnimg python_series/install_python_2.png %}

安装之后可能程序没有自动配置anaconda环境变量，你需要手动配置！！！

找到刚才安装的anaconda的目录，找到Scripts，打开，复制路径：

路径示例：D:\ProgramData\Anaconda3\Scripts。

配置环境变量：

在Path后面添加刚才复制的路径，注意与前一个要用英文分号隔开。点击多个确定完成配置。

打开cmd

输入python，看到下面的画面，说明安装成功。

<!-- ![](/images/articles/2019/python_series/install_python_3.png) -->
{% qnimg python_series/install_python_3.png %}

看到提示符>>>就表示我们已经在Python交互式环境中了，可以输入任何Python代码，回车后会立即得到执行结果。输入`exit()`并回车，就退出Python交互模式。



# 第一个Python程序
---
## Python交互模式

在Python交互式环境中输入以下代码
```python
print('Hello world!')
```

<!-- ![](/images/articles/2019/python_series/install_python_4.png) -->
{% qnimg python_series/install_python_4.png %}

## 命令行模式

通过`python  xxx.py`运行一个`.py`文件

创建一个hello.py文件，文件中输入如下代码：
```python
print('Hello World!!!')
```

cmd的当前目录切换到`hello.py`所在的目录下，我的目录在 `E:\workspace\python-learning\1-环境搭建` ，执行下面命令可以进入到相应目录中。

```
C:\Users\chenhp>E:

E:\>cd workspace

E:\workspace>cd python-learning

E:\workspace\python-learning>cd 1-环境搭建
```

在命令行中执行`python hello.py`，可以得到下面的执行结果：

<!-- ![](/images/articles/2019/python_series/install_python_5.png) -->
{% qnimg python_series/install_python_5.png %}


以上内容主要介绍了windows上的环境搭建。关于macOS的环境搭建，可以百度一下安装教程，作为一个没用过mac的人就不在这里就瞎掰了。

**文中示例代码**： [python-learning](https://github.com/HamptonChen/python-learning)

未完待续，持续更新中......