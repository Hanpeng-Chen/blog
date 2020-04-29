---
title: Python从小白到攻城狮(5)——深入浅出字符串
urlname: python-5-string
tags:
  - python教程
  - 字符串
categories:
  - Python
abbrlink: 27490
date: 2019-08-25 22:54:02
---

在[《Python从小白到攻城狮（2）：数据类型和变量》](http://www.chenhanpeng.com/2019/08/06/python_series/Python从小白到攻城狮（2）：数据类型和变量/)中，我们简单介绍过字符串，今天这篇文章，我们将一起学习字符串的更多知识。

# 字符串基础
---
字符串(string)是Python中很常见的一种数据类型，在日志的打印、函数的注释、数据库的访问、变量的基本操作等等中都用到了字符串。

字符串是以单引号(`'`) 、双引号(`"`) 或者三引号(`'''` 或 `"""`)括起来的任意文本。

在python中单引号、双引号、三引号的字符串是一模一样的，没有区别。

我们来看一下字符串的几种写法：
```python
name = 'zhang san'
city = "Fujian"

s1 = 'hello world'
s2 = "hello world"
s3 = """hello world"""
print(s1 == s2 == s3) # 返回True说明s1 s2 s3完全一样
```

Python的字符串同时支持三种表达方式，主要是方便我们在字符串中，内嵌带引号的字符串。比如：
```python
"I'm a coder"
"hello 'world'"
```

三引号字符串，主要应用于多行字符串的情境，比如函数的注释等等。
```python
def function_notes(value1, value2):
  """
  args:
    value1: number
    value2: number
  return
    value1 + value2
  """
  return value1 + value2
```

Python也支持转义字符。

转义字符：用反斜杠开头的字符串，来表示一些特定意义的字符。常见的转义字符见下表：

<!-- ![](/images/articles/2019/python_series/5-1.png) -->
{% qnimg python_series/5-1.png %}

```python
>>> s = 'a\nb\tc\''
>>> print(s)
a
b       c'

>>> print(len(s))
6
```

虽然字符串s打印的输出横跨了两行，但是整个字符串s仍然只有6个元素。

在转义字符中，最常见的是换行符`\n`的使用。在文件读取时，如果我们一行一行读取，那么每行字符串的末尾，都会包含换行符`\n`。在最后进行数据处理时，我们往往会去掉每一行的末尾的换行符。


# 字符串的常用操作
---
Python的字符串支持索引、切片和遍历等操作。

和其他数据结构一样，字符串的索引也是从0开始，index=0表示第一个元素（字符），[index:index+2]表示第index个元素到index+1个元素组成的子字符串。
```python
text = 'welcom to china'
print(text[0])
print(text[1:4])
```

遍历字符串同样很简单，相当于遍历字符串中的每个字符
```python
text = 'welcom to china'
for char in text:
  print(char)
```

特别要注意，Python的字符串是不可变的（immutable）。Python中字符串的改变，通常只能通过创建新的字符串来完成。比如：我们想把`'hello'`的第一个字符`'h'`，改为大写的`'H'`，可以采用下面的做法：
```python
s = 'H' + s[1:]
s = s.replace('h', 'H')
```
- 第一种方法，是直接用大写的`'H'`，通过加号`'+'`操作符，与原字符串切片操作的子字符串拼接而成新的字符串。
- 第二种方法，是直接扫描原字符串，把小写的`'h'`替换成大写的`'H'`，得到新的字符串。

字符串常用的函数还有下面几个：

- **string.strip(str)**：表示去掉首尾的str字符串
- **string.lstrip(str)**：表示只去掉开头的str字符串
- **string.rstrip(str)**：表示只去掉尾部的str字符串
- **string.find(sub, start, end)**：表示从start到end查找字符串中子字符串sub的位置等等
- **string.split(separator)**：表示把字符串按照separator分割成子字符串，并返回一个分割后子字符串组合的列表


下面我们通过一些练习来熟悉字符串的操作：
```python
str1 = "hello World!"
# 利用len函数计算字符串长度
print(len(str1)) # 12

# 获取字符串首字母大写的拷贝
print(str1.capitalize()) # Hello world!

# 获取字符串变大写后的拷贝
print(str1.upper()) # HELLO WORLD!

# find函数查找子串的位置
print(str1.find('llo')) # 2
print(str1.find('hot')) # -1
# # index查找子串，但找不到子串或报错
# print(str1.index('llo'))
# print(str1.index('hot'))

# 判断字符串是否以指定的字符串开头
print(str1.startswith('he')) # True
print(str1.startswith('ha')) # False

# 判断字符串是否以指定的字符串结尾
print(str1.endswith('d!')) # True
print(str1.endswith('ld')) # False

# 将字符串以指定的宽度居中并在两侧填充指定的字符
print(str1.center(50, '='))
# 将字符串以指定的宽度靠右放置左侧填充指定的字符
print(str1.rjust(50, '-'))

str2 = '1234abcd'
# 索引操作
print(str2[3]) # 4
# 字符串切片操作(从指定的位置开始索引)
print(str2[2:5])  # 34a
print(str2[2:])  # 34abcd
print(str2[2::2])  # 3ac
print(str2[::2])  # 13ac
print(str2[::-1])  # dcba4321
print(str2[-3:-1])  # bc

# 检查字符串是否由数字构成
print(str2.isdigit())  # False
# 检查字符串是否以字母构成
print(str2.isalpha())  # False
# 检查字符串是否以数字和字母构成
print(str2.isalnum())  # True

str3 = '  my name is zhangsan  '
# 获得字符串修剪头尾空格后的拷贝
print(str3.strip())  # my name is zhangsan
```


# 字符串的格式化
---
## 什么是字符串的格式化？
通常，我们使用一个字符串作为模板，模板中会有格式符。这些格式符为后续真实值预留位置，以呈现出真实值应该呈现的样式。
字符串的格式化，通常会用在程序的输出、logging等场景。

比如：我们有一个任务，给定一个用户的userid，要去数据库中查询该用户的一些信息，并返回。如果数据库中没有此人的信息，我们通常会记录下来，这样有利于往后的日志分析，或者是线上bug的调试等。
```
id = 123456
nanme = 'zhangsan'
print('no data available for person with id: {}, name: {}'.format(id, name))
```

其中`string.format()`就是格式化函数，大括号{}就是所谓的格式符，用来为后面的真实值————变量id、name预留位置。


不过要注意，string.format()是最新的字符串格式函数与规范。自然，我们还有其他的表示方法，比如在Python之前版本中，字符串格式化通常用%来表示，那么上述的例子，就可以写成下面这样：
```python
print('no data available for person with id: %s, name: %s' % (id, name))
```

其中%s表示字符串型，%d表示整型等等，这些属于常识，你应该都了解。

当然，现在你写程序时，我还是推荐使用format函数，毕竟这是最新规范，也是官方文档推荐的规范。

也许有人会问，为什么非要使用格式化函数，上述例子用字符串的拼接不也能完成吗？没错，在很多情况下，字符串拼接确实能满足格式化函数的需求。但是使用格式化函数，更加清晰、易读，并且更加规范，不易出错。


**文中示例代码**： [python-learning](https://github.com/Hanpeng-Chen/python-learning)

未完待续，持续更新中......