---
title: Python从小白到攻城狮（7）：函数
date: 2019-09-05 15:55:35
tags:
  - python教程
  - 函数
categories:
  - Python
---
# 什么是函数
---
函数是组织好的，可复用的，用来实现单一，或相关联功能的代码段。

函数能提高应用的模块性和代码的复用率。在编程中，常将一些常用的功能写成函数放在函数库中供公共选用。利用好函数，可以减少我们重复编码的工作。

在前面学习中，我们用过许多Python的内置函数，比如print()、sort()等。

# 如何定义一个函数
---
根据下面的几个规则，我们可以定义一个函数：
* 函数代码块以def关键词开头，后接函数标识符名称和圆括号()。
* 任何传入参数和自变量必须放在圆括号中。圆括号之间可以用于定义参数。
* 函数的第一行语句可以选择性地使用文档字符串——用于存放函数说明。
* 函数内容以冒号起始，并且缩进。
* return [表达式] 结束函数，选择性地返回一个值给调用方。不带表达式的return相当于返回None。也可以不返回。

**语法**
```python
def function_name(parameters):
    函数体
    return [表达式]
```
默认情况下，参数值和参数名称是按函数声明中定义的顺序匹配起来的。

# 简单的示例
---
```python
# 定义一个简单的函数
def print_str():
    print('hello world!')

print_str()

# 定义一个带参数的函数
def print_string(str):
    print(str)

print_string('Hello World!!!!')
```

# 返回多个值
---
Python的函数支持返回多个值，我们通过下面的例子来了解其用法：
```python
# 返回多个值
def test(x, y):
    x1 = x + 2
    y1 = y + 3
    return x1, y1

x, y = test(10, 20)
print(x, y)
```

# 函数的参数
---
调用函数时可使用的正式参数类型有以下4种：必备参数、关键字参数、默认参数、不定长参数。

## 必备参数
必备参数要以正确的顺序传入函数，调用时的数量必须和声明时的一样。

下面例子中的print_necessary_params()函数，调用时必须传入一个参数，不然会出现语法错误。
```python
def print_necessary_params(str1):
    print(str1)

print_necessary_params()

######控制台输出
Traceback (most recent call last):
  File "function.py", line 28, in <module>
    print_necessary_params()
TypeError: print_necessary_params() missing 1 required positional argument: 'str1'
```

## 关键字参数
关键字参数和函数调用关系紧密，函数调用使用关键字参数来确定传入的参数值。

使用关键字参数允许函数调用时参数的顺序与声明时不一致，因为 Python 解释器能够用参数名匹配参数值。
```python
def print_keyword_params(id, price):
    print('id:', id)
    print('price', price)

print_keyword_params(price=20, id=125345)
```

## 默认参数
调用函数时，默认参数的值如果没有传入，则被认为是默认值。
```python
def print_default_param(price = 50):
    print(price)

print_default_param()
print_default_param(20)
```

## 不定长参数
你可能需要一个函数能处理比当初声明时更多的参数。这些参数叫做不定长参数，和上述2种参数不同，声明时不会命名。基本语法如下：
```python
def functionname([formal_args,] *var_args_tuple ):
   "函数_文档字符串"
   function_suite
   return [expression]
```
加了星号（*）的变量名会存放所有未命名的变量参数。不定长参数实例如下：
```python
def print_info(arg1, *params):
    print('arg1:', arg1)
    for var in params:
        print(var)

print_info('test')
print_info(10, 20, 30, 40)
```

# 递归函数
---
有时候我们需要反复调用某个函数得到一个最后的值，这个时候使用递归函数是最好的解决方案。

编程语言中，函数Func(Type a,……)直接或间接调用函数本身，则该函数称为递归函数。递归函数不能定义为内联函数。

举个例子，我们来计算阶乘n! = 1 x 2 x 3 x ... x n，用函数fact(n)表示，可以看出：

fact(n) = n! = 1 x 2 x 3 x ... x (n-1) x n = (n-1)! x n = fact(n-1) x n

所以，fact(n)可以表示为n x fact(n-1)，只有n=1时需要特殊处理。

于是，fact(n)用递归的方式写出来就是：
```python
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)

print(fact(5))
print(fact(10))
```

# 总结
本节介绍了Python函数的定义，不同类型的参数的使用以及递归函数的用法。

**文中示例代码**： [python-learning](https://github.com/HamptonChen/python-learning)

未完待续，持续更新中......


参考：
https://www.runoob.com/python/python-functions.html
https://www.liaoxuefeng.com/wiki/1016959663602400/1017268131039072
http://www.ityouknow.com/python/2019/08/08/python-005.html