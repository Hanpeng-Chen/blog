---
title: Python从小白到攻城狮(11)——输入与输出
urlname: python-11-input-and-output
date: 2019-09-23 22:49:27
tags:
  - python教程
categories:
  - Python
---
在前面章节中，我们已经用过Python的输入和输出功能，本节将具体介绍Python的输入和输出。

# 输入输出
---
Python中最简单直接的输入来自键盘操作，我们先来看下面的例子：
```python
name = input('your name is:')
age = input('your age:')
print('name: ' + name)
print('age: ' + age)
##########控制台输入与输出内容###########
your name is:Jack
your age:20
name: Jack
age: 2
```

input()函数：暂停程序运行，同时等待键盘输入，知道回车键被按下，函数的参数即为提示语，**输入的类型永远都是字符串类型**。

print()函数：接受字符串、数字、字典、列表甚至一些自定义类的输出。

我们来看下面的例子：

```python
a = input() # 输入1
b = input() # 输入2

print('a + b = {}'.format(a + b))
print('type of a is {}, type of b is {}'.format(type(a), type(b)))
print('a + b = {}'.format(int(a) + int(b)))
#######输出########
a + b = 12
type of a is <class 'str'>, type of b is <class 'str'>
a + b = 3
```

str强制转换为int用int()，转为浮点数用float()。在生产环境中使用强制转换，记得加上try  except对错误或异常进行处理。

在Python中，int类型没有最大限制，float类型有精度限制。

# 文件输入输出
## open()
open()将会返回一个file对象，基本语法格式如下：
```python
open(filename, mode)
```
> filename：包含你要访问的文件名称的字符串值。
> mode：决定了打开文件的模式：只读，写入，追加等。这个参数非强制的，默认文件访问模式为只读(r)。所有可取值见下表：

![](/images/articles/2019/python_series/11-1.png)

以下示例将字符串写入到文件demo.txt中：
```python
# 打开一个文件
f = open('./file_1.txt', 'w')
f.write('Python是一门有用的编程语言。\nHello world！')
# 关闭打开的文件
f.close()
```

查看file_1.txt文件，内容如下：
```
Python是一门有用的编程语言。
Hello world！
```

## 文件对象的方法
假设已经创建了一个文件对象f

### f.read()
调用f.read(size)方法，将读取文件的一定数目的数据，然后作为字符串或字节对象返回。

size是一个可选的数字类型的参数。当size被忽略或为负时，那么该文件的所有内容都将被读取并返回。

读取上一个例子创建的file_1.txt的内容：
```python
f = open('./file_1.txt', 'r')
str = f.read()
print(str)
f.close()
```

### f.readline()
f.readline()会从文件中读取单独的一行。换行符为"\n"。f.readline()如果返回一个空字符串，说明已经读取到最后一行。

```python
f = open('./file_1.txt', 'r')
str = f.readline()
print(str)
str1 = f.readline()
print(str1)
str2 = f.readline()
print(str2)
f.close()
```

### f.readlines()
f.readlines()将返回该文件中包含的所有行。

如果设置可选参数sizehint，则读取指定长度的字节，并且这些字节按行分割。
```python
f = open('./file_1.txt', 'r')
str = f.readlines()
print(str)  # ['Python是一门有用的编程语言。\n', 'Hello world！']
f.close()
```

### f.write()
f.write(string)将string写入到文件中，然后返回写入的字符数。

如果要写入一些不是字符串的东西，那么需要先进行转换：
```python
f = open('./file_2.txt', 'w')
value = ('Python从小白到攻城狮', 666)
s = str(value)
f.write(s)
f.close()
```

### f.tell()
f.tell()返回文件对象当前所处的位置，它是从文件开头开始算起的字节数。

### f.seek()
如果要改变文件当前的位置，可以使用f.seek(offset, from_what)函数。

from_what的值默认为0，如果是0表示开头，如果是1表示当前位置，2表示文件的结尾，例如：
* seek(x, 0)：从起始位置即文件首行首字符开始移动x个字符
* seek(x, 1)：表示从当前位置往后移动x个字符
* seek(-x, 2)：表示从文件的结尾往前移动x个字符
```python
>>> f = open('./file_2.txt', 'rb+')
>>> f.write(b'0123456789abcdef')
16
>>> f.seek(5)
5
>>> f.read(1)
b'5'
>>> f.seek(-3, 2)
26
>>> f.read(1)
b'6'
```


### f.close()
在文本文件中 (那些打开文件的模式下没有 b 的), 只会相对于文件起始位置进行定位。

当你处理完一个文件后, 调用 f.close() 来关闭文件并释放系统的资源，如果尝试再调用该文件，则会抛出异常。

当处理一个文件对象时, 使用 with 关键字是非常好的方式。在结束后, 它会帮你正确的关闭文件。 而且写起来也比 try - finally 语句块要简短。


# JSON序列化
---
JSON(JavaScript Object Notation)是一种轻量级的数据交换格式。其设计意图是把所有事情都用设计的字符串来表示，既方便在互联网上传递信息，也方便人阅读。在当今互联网中，JSON的应用非常广泛。

* json.dumps()函数：接受Python的基本数据类型，然后将其序列化为string。
* json.loads()函数：接受一个合法字符串，然后将其反序列化为Python的基本数据类型。

```python
import json

data = {
  'name': 'Jack',
  'age': 20,
  'address': 'Fujian, China',
}

data_str = json.dumps(data)
print('after json dumps')
print('type of data_str is {}, data_str = {}'.format(type(data_str), data_str))

origin_data = json.loads(data_str)
print('after json loads')
print('type of origin_data is {}, origin_data = {}'.format(type(origin_data), origin_data))
```

**文中示例代码**： [python-learning](https://github.com/HamptonChen/python-learning)

未完待续，持续更新中......