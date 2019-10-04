---
title: Python从小白到攻城狮(10)——高阶函数
urlname: python-10-higher-order-function
date: 2019-09-22 21:28:53
tags:
  - python教程
  - 高阶函数
categories:
  - Python
---

本节将主要介绍什么是高阶函数、高阶函数的用法以及Python的几个常见的内置高阶函数。

# 什么是高阶函数

高阶函数(Higher-order function)：一个函数可以作为参数传给另一个函数，或者一个函数的返回值为另一个函数（若返回值为该函数本身，则为递归），满足其一则为高阶函数。

下面，我们通过例子来进一步学习高阶函数的用法。

**参数为函数**
```python
# 参数为函数
def print_hello():
    print('hello world')

def print_function(func):
    func()
    print('in the print_function')

print_function(print_hello)

########输出结果######
hello world
in the print_function
```
在上面的代码中，我们将`print_hello()`函数作为参数func传递给print_function()函数，直接调用print_hello()和调用func()结果一样。

**返回值为函数**
```python
# 返回值为函数
def print_hello():
    print('hello world')

def print_function():
    print('in the print_func')
    return print_hello

result = print_function()
result()

#####输出#####
in the print_func
hello world
```
上面的例子中，print_hello()函数作为print_function()的返回值。

> 备注：函数名(print_hello、print_function)-->其为该函数的内存地址；函数名+括号(print_hello())-->调用该函数。

# Python内置的高阶函数

map()、filter()、reduce()、sorted()，这几个均为Python内置的高阶函数。通常结合匿名函数lambda一起使用。接下来我们看一下这三个函数的用法以及其内部原理。

## map(function, iterable)函数：
map函数接收的是两个参数：一个函数，一个Iterable。map对iterable中的每个元素，都运用function这个函数，最后返回一个新的Iterable，即迭代器对象，例如：`<map object at 0x00000214EEF40BA8>`。

比如：要对列表中的每个元素乘以2，用map可以实现如下：
```python
l = [1, 2, 3, 4, 5]
re = map(lambda x: x * 2, l)
print(list(re))  # [2， 4， 6， 8， 10]
```


## filter(function, iterable)函数
filter函数和map函数类似，也是接收一个函数和一个序列的高阶函数，主要功能是过滤。

filter()函数表示对iterable中的每个元素，都是用function判断，并返回True或False，最后将返回True的元素组成一个迭代器对象，其返回值也是迭代器对象，例如：`<filter object at 0x000002042D25EA90>`。

注意到filter()函数返回的是一个Iterator，也就是一个惰性序列，所以要强迫filter()完成计算结果，需要用list()函数获得所有结果并返回list。

比如：我们要返回一个列表中的所有奇数：
```python
l = [1, 2, 3, 4, 5]
result = filter(lambda x: x % 2 == 1, l)
print(list(result))  #[1, 3, 5]
```

## reduce(function, iterable)函数
这里的function同样是一个函数对象，规定它有两个参数，表示对iterable中的每个元素及上一次调用的结果，运用function进行计算，所以最后返回的是一个单独的数值。

例子：计算某个列表元素的乘积。
```python
from functools import reduce

l2 = [1, 2, 3, 4, 5]
result = reduce(lambda x, y: x * y, l2)
print(result)  # 1*2*3*4*5=120
```

>python3中reduce函数被取消了，放入到了functools模块中，所以在语句前加上一条：`from functools import reduce`


## sorted()函数
sorted() 函数对所有可迭代的对象进行排序操作。

**sort 与 sorted 区别**：
- sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。
- list 的 sort 方法返回的是对已经存在的列表进行操作，而内建函数 sorted 方法返回的是一个新的 list，而不是在原来的基础上进行的操作。

**sorted语法**：
sorted(iterable, key=None, reverse=False)  

**参数说明**：
* iterable：可迭代对象
* key：主要用来进行比较的元素，只有一个参数，具体的函数的参数就是取自于可迭代对象中，指定可迭代对象中的一个元素来进行排序。
* reverse：排序规则，reverse=True 降序，reverse=False 升序（默认）。

**返回值**
返回重新排序的列表

**示例**
```python
a = [1, 5, 3, 6, 9, 8]
b = sorted(a)
print(a) # [1, 5, 3, 6, 9, 8]
print(b)   # [1, 3, 5, 6, 8, 9]


l = [('b', 2), ('a', 1), ('c', 3), ('d', 4)]
l1 = sorted(l, key=lambda s:s[1])
print(l1)   # [('a', 1), ('b', 2), ('c', 3), ('d', 4)]
l2 = sorted(l, key=lambda s:s[1], reverse=True) # 降序
print(l2)   # [('d', 4), ('c', 3), ('b', 2), ('a', 1)]
```

**文中示例代码**： [python-learning](https://github.com/HamptonChen/python-learning)

未完待续，持续更新中......