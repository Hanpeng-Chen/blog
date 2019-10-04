---
title: Python从小白到攻城狮(6)——条件与循环
urlname: python-6-condition-and-loop
date: 2019-08-27 14:12:08
tags:
  - python教程
  - 条件与循环
categories:
  - Python
---

在前面几篇文章中，我们学习了列表、元组、字典、集合和字符串等一系列Python的基本数据类型和数据结构。但仅靠这些数据结构类型是无法支持整个程序运行的，在编程中，流程控制是程序运行的基础，它决定了程序按照什么方式去执行。

接下来给大家介绍Python流程控制相关语法。

# 条件语句
---
Python的条件语句和其他语言一样，都是用`if`语句实现。

if语句表示如果发生什么样的条件，执行什么样的逻辑。

语法如下：
```python
if condition_1:
    statement_1
elif condition_2:
    statement_2
...
elif condition_i:
    statement_i
else:
    statement_n
```
示例：
```python
id = 1
if id == 0:
  print('red')
elif id == 1:
  print('blue')
else:
  print('yellow')
```

由于Python不支持switch语句，因此，当存在多个条件判断时，我们需要用else if来实现，这在Python中的表达是 **elif**。在条件语句中，可能会有零到多个elif部分，else是可选的。关键字 'elif' 是 'else if' 的缩写，这个可以有效地避免过深的缩进。

整个条件语句是顺序执行的，如果遇到一个条件满足，比如condition_i满足时，再执行完statement_i后，便会退出整个条件语句，而不会继续向下执行。


需要注意的是：

* 在条件语句末尾必须加上冒号(:)，这是Python特定的语法规范。
* if语句可以单独使用，但elif、else都必须和if成对使用。

# 循环语句
循环，本质上就是遍历集合中的元素。和其他语言一样，python中的循环一般通过for循环和while循环实现。

## for循环
语法格式如下：
```python
'''
for 后面跟变量名，in后面跟序列（主要指列表、元组、字符串、文件等等）
for 循环每次从序列中取一个值放到变量中
'''
for item in <iterable>:
    statements(s)
```
示例：
```python
s = 'hello world'
for char in s:
  print(char)

list = [1, 2, 3, 4, 5]
for item in list:
  print(item)
```

我们也可以通过索引来遍历元素。通常通过**range()**函数拿到索引，再去遍历访问元素。

```
# 生成一个等差级数链表
range (start， end， scan):
```
参数含义：
* start：计数从start开始，默认从0开始。比如range(3)等价于range(0, 3)
* end：计数到end结束，但不包括end。比如range(0, 3)的结果是[0, 1, 2]，但不包含3
* scan：每次跳跃的间距，默认为1。

示例如下：

```python
# 通过range()函数获取索引，再去遍历
l = ['zhangsan', 'lisi', 'wangwu', 'sunliu', 'zhouqi']
for index in range(0, len(l)):
  print(l[index])
```

如果我们同时需要索引和元素时，可以通过Python内置的enumerate()函数来遍历集合。
```python
# enumerate()函数来遍历集合
for index, item in enumerate(l):
  print('index: {}, value: {}'.format(index, item))
```

这里单独强调一下字典。字典本身只有键是可迭代的，如果我们要遍历它的值或者键值对，需要通过其内置的函数values()或items()实现。其中，values()返回字典的值的集合，items()返回键值对的集合。

```python
d = {'name': 'jack', 'age': 20, 'gender': 'male'}
# 遍历字典的键
for k in d:
  print(k)


# 遍历字典的值
for val in d.values():
  print(val)

# 遍历字典的键值对
for k, v in d.items():
  print('key: {}, value: {}'.format(k, v))
```

## while循环
while循环，表示当condition满足时，一直重复循环执行某段程序，直到condition不再满足，就跳出循环体。
```
while condition:
    执行语句.....
```

很多时候，for循环和while循环可以相互转换，比如要遍历一个列表，用while循环同样可以完成。
```python
list = [1, 2, 3, 4, 5]
index = 0
while index < len(list):
  print(list[index])
  index += 1
```

也可以在 while 循环中添加判断逻辑
```python
count = 0
while count < 3:
  print('count 小于3')
  count += 1
else:
  print('count 大于等于3')
```

## break用法
break可以跳出for循环和while循环。如果从for或while循环跳出，相应的循环else代码块将不执行。
示例：
```python
for char in 'hello world':
  if char == 'l':
    break
  print(char)


count = 0
while count < 10:
  if count >= 5:
    break
  print(count)
  count += 1
```

## continue用法
continue语句用来跳过当前循环块中的剩余语句，然后进行下一轮循环。
```python
i = 1
while i < 10:   
    i += 1
    if i%2 > 0:     # 非双数时跳过输出
        continue
    print(i)
```

# 条件与循环的复用
前面我们介绍了条件与循环的基本操作，接下来我们来看看它们的进阶操作，让程序变得更简洁高效。

在阅读别人代码时，我们会发现，很多将条件和循环并做一行的写法，如：
```
expression1 if condition else expression2 for item in iterable
```
我们将上面的表达式分解成多行代码，等同于下方的嵌套结构：
```python
for item in iterable:
    if condition1:
        expression1
    else:
        expression2
```

如果表达式中没有else语句，则需要写成：
```python
expression1 for item in iterable if condition1
```

接下来我们用两个例子来熟悉一下这种写法。

1、我们要绘制y = 2*|x| + 5 的函数图像，给定集合x的数据点，需要计算出y的数据集合
```python
x = [-2, -1, 0, 1, 2]
y = [value * 2 + 5 if value > 0 else -value * 2 + 5 for value in x]
print(y)
```

2、将文件中逐行读取的一个完整语句，按逗号分割单词，去掉首位的空字符，并过滤掉长度小于等于5的单词，最后返回由单词组成的列表
```python
text = ' Today , is, Sunday '
text_list = [s.strip() for s in text.split(',') if len(s) > 5]
print(text_list)
```

当然，这样的复用并不仅仅局限于一个循环。比如：给定两个列表x、y, 要求返回x、y中所有元素对组成的元组，相等的情况除外。那么，我们可以用下面的表达式表示出来：
```python
[(xx, yy) for xx in x for yy in y if xx != yy]
```
写法等价于：
```python
l = []
for xx in x:
    for yy in y:
        if xx != yy:
            l.append((xx, yy))
```

熟练之后，你会发现这种写法非常方便。当然，如果遇到逻辑很复杂的复用，你可能会觉得写成一行难以理解、容易出错。那种情况下，用正常的形式表达，也不失为一种好的规范和选择。

下面的一个练习题，大家尝试分别用一行和多行条件循环语句来实现，参考代码点击文末示例代码链接获取。
```
给定下面两个列表attributes和values，要求针对values中每一组子列表value，输出其和attributes中的键对应后的字典，最后返回字典组成的列表。
attributes = ['name', 'dob', 'gender']
values = [['jason', '2000-01-01', 'male'], 
['mike', '1999-01-01', 'male'],
['nancy', '2001-02-01', 'female']
]

# expected output:
[{'name': 'jason', 'dob': '2000-01-01', 'gender': 'male'}, 
{'name': 'mike', 'dob': '1999-01-01', 'gender': 'male'}, 
{'name': 'nancy', 'dob': '2001-02-01', 'gender': 'female'}]
```

# 总结

- 在条件语句中，if可以单独使用，但是elif和else必须和if同时搭配使用；而If条件语句的判断，除了boolean类型外，其他的最好显示出来。
- 在for循环中，如果需要同时访问索引和元素，你可以使用enumerate()函数来简化代码。
- 写条件与循环时，合理利用continue或者break来避免复杂的嵌套，是十分重要的。
- 要注意条件与循环的复用，简单功能往往可以用一行直接完成，极大地提高代码质量与效率。

**文中示例代码**： [python-learning](https://github.com/HamptonChen/python-learning)

未完待续，持续更新中......