---
title: Python从小白到攻城狮(13)——迭代器
tags:
  - python教程
  - 迭代器
urlname: python-13-iterator
categories:
  - Python
date: 2019-10-10 00:36:00
---
# 迭代器
迭代是Python最强大的功能之一，是访问集合元素的一种方式。

迭代器是一个可以记住遍历的位置的对象。

迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退。

迭代器有两个基本的方法：`iter()` 和 `next()`。

字符串、列表和元组对象都可用于创建迭代器。

示例：
```python
list = [1, 2, 3, 4]
it = iter(list)
print(next(it))
print(next(it))
```

迭代器对象可以使用常规for语句进行遍历：
```python
list = [1, 2, 3, 4]
it = iter(list)
for x in it:
    print(x, end=",\n")
```

也可以使用next()函数：
```python
import sys

list = [1, 2, 3, 4]
it = iter(list)

while True:
    try:
        print(next(it))
    except StopIteration:
        sys.exit()
```

## 创建一个迭代器
把一个类作为一个迭代器使用需要在类中实现两个方法：`__iter__()` 和 `__next__()`

**__iter__()**：返回一个特殊的迭代器对象，这个迭代器对象实现了__next__() 方法，并通过StopIteration异常标识迭代的完成。

**__next__()**：返回下一个迭代器对象。

通过下面的例子，看一下如何创建一个迭代器：
```python
# 创建一个返回数字的迭代器，初始值为0，逐步递增10
class numbers:
    def __iter__(self):
        self.a = 0
        return self

    def __next__(self):
        x = self.a
        self.a += 10
        return x

myclass = numbers()
it = iter(myclass)

print(next(it))
print(next(it))
print(next(it))
print(next(it))
```

## StopIteration
**StopIteration异常用于标识迭代的完成，防止出现无限循环的情况**，在 __next__() 方法中我们可以设置在完成指定循环次数后触发 StopIteration 异常来结束迭代。

把上面创建迭代器的例子拿来修改，当值大于50时，停止迭代。
```python
# 创建一个返回数字的迭代器，初始值为0，逐步递增10
class numbers:
    def __iter__(self):
        self.a = 0
        return self

    def __next__(self):
        if self.a > 50:
            raise StopIteration
        else:
            x = self.a
            self.a += 10
            return x

myclass = numbers()
it = iter(myclass)

for x in it:
    print(x)
```


**文中示例代码**： [python-learning](https://github.com/HamptonChen/python-learning)

未完待续，持续更新中......