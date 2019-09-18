---
title: Python从小白到攻城狮(8)——模块
date: 2019-09-05 23:00:13
author: 陈汉鹏
tags:
  - python教程
  - 模块
categories:
  - Python
---
随着程序代码越写越多，一个文件中的代码越来越长，也越来越难以维护。为了编写可维护的代码，我们把很多函数分组，放到不同的文件中，这样每个文件包含的代码相对较少。

在Python中，一个.py文件就称为一个模块(Module)，里面定义了一些函数、类和变量，也可能包含可执行的代码。

模块让你能够有逻辑地组织你的 Python 代码段。把相关的代码分配到一个模块里能让你的代码更好用，更易懂。

# 模块
---
从上面我们知道了模块就是一个.py文件，接下来我们来看下如何定义和引用一个模块。

## 如何定义一个模块
下面是简单的模块hello.py:
```python
def print_hello():
    print('hello world!')
```

## import 语句
定义好模块后，我们可以使用import语句来引入模块，语法如下：
```python
import module1[, module2[,... moduleN]]
```
当解释器遇到import语句，如果模块在当前的搜索路径就会被导入。

调用的时候使用**函数名.方法名**来进行调用。

我们新建一个test_hello.py文件来调用hello.py模块的方法。
```python
import hello

hello.print_hello()
```
不管你执行多少次import，一个模块只会被导入一次。这样可以防止导入模块被一遍又一遍地执行。

## from...import语句
Python的from语句让你从模块中导入一个指定的部分到当前命名空间中。语法如下
```
from module import name1[, name2[, ... nameN]]
```
例如，我们要导入hello.py模块中的print_hello方法，使用如下语句：
```
from hello import print_hello
```
这个声明不会把整个 hello 模块导入到当前的命名空间中，它只会将 hello 里的 print_hello 单个引入到执行这个声明的模块的全局符号表。

## from…import* 语句
把一个模块的所有内容全都导入到当前的命名空间也是可行的，只需使用如下声明：
```
from modname import *
```
>这提供了一个简单的方法来导入一个模块中的所有项目。然而这种声明不该被过多地使用。


# Python中的包
---
包是一个分层次的文件目录结构，它定义了一个由模块及子包，和子包下的子包等组成的Python的应用环境。

简单来说，包就是文件夹，但该文件夹下必须存在__init__.py文件，该文件的内容可以为空。__init__.py用于标识当前文件夹是一个包。

示例：
我们创建一个test包，目录结构如下：
```
test
 -- __init__.py
 -- test_cal.py
```

test_cal.py模块的代码如下：
```python
def add(x, y):
  return x + y

def square(x):
  return x * x
```

## 包的使用
Python包的使用和模块的使用类似，可以通过import语句和from...import语句导入。

新建一个do.py来导入和使用test包，我们先用import语句导入：
```python
import test.test_cal
print(test.test_cal.add(10, 20))
```

如果包含多层子包，那么调用方法前面还有写一长串的包名，特别不方便，我们可以用from...import语句来简化：
```python
from test.test_cal import square
print(square(10))
```


**文中示例代码**： [python-learning](https://github.com/HamptonChen/python-learning)

未完待续，持续更新中......


参考：
https://www.runoob.com/python/python-modules.html