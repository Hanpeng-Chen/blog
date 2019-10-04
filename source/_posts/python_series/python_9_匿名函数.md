---
title: Python从小白到攻城狮(9)——匿名函数
urlname: python-9-lambda
date: 2019-09-18 12:00:13
tags:
  - python教程
categories:
  - Python
---
在前面函数那节中，我们一起学习了Python的常规函数。但是在代码中，除了常规函数，我们也会见到一些“非常规”函数，它们往往很简短，就一行，并且有个很酷炫的名字——`lambda`，这就是匿名函数。

# 什么是匿名函数
所谓匿名，即不再使用def语句这样标准的形式定义一个函数。

* lambda只是一个表达式，函数体比def简单得多。
* lambda的主题是一个表达式，而不是一个代码块，仅仅能在lambda表达式中封装有限的逻辑进去。
* lambda函数拥有自己的命名空间，且不能访问自己参数列表之外或全局命名空间里的参数。
* 虽然lambda函数看起来只有一行，却不等同于C或C++的内联函数，后者的目的是调用小函数时不占用栈内存从而增加运行效率。

# 语法
lambda函数的语法只包含一个语句，如下：
```
lambda argument1, argument2,... argumentN : expression
```

从上面的语法我们可以看到，匿名函数的关键字是lambda，之后是一系列的参数，用冒号隔开，最后则是由这些参数组成的表达式。

我们通过几个例子来看下它的用法：
```python
square = lambda x: x**2
print(square(5))

sum = lambda x, y: x + y
print(sum(2, 3))
```

可以看到，匿名函数lambda和常规函数一样，返回的都是一个函数对象(function object)，它们的用法也极其相似，不过有下面几点区别：

# 匿名函数和常规函数区别

## 第一，lambda是一个表达式(expression)，而不是一个语句(statement)。
* 所谓的表达式，就是用一系列“公式”去表达一个东西，比如 x+y 、 x**2等等。
* 所谓的语句，则一定是完成了某些功能，比如赋值语句 x = 2完成了赋值，print语句print(x) 完成了打印等等。

因此，lambda可以用在一些常规函数def不能用的地方，比如，lambda可以用在列表内部，而常规函数却不能：
```python
l = [(lambda x: x*x)(x) for in range(10)]
print(l)
```

再比如，lambda可以被用作某些函数的参数，而常规函数def也不能：
```python
l = [(1, 20), (3, 10), (5, 25), (2, 0)]
l.sort(key=lambda x: x[1]) # 按照列表中元组的第二个元素排序
print(l)
```

常规函数def必须通过其函数名被调用，因此必须首先被定义。但是作为一个表达式的lambda，返回的函数对象就不需要名字了。

## 第二，lambda的主体是只有一行的简单表达式，并不能扩展成一个多行的代码块。

这其实是出于设计的考虑。Python之所以发明lambda，就是为了让它和常规函数各司其职：lambda专注于简单的任务，而常规函数则负责更复杂的多行逻辑。

用匿名函数改造下面的代码：
```
def is_odd(n):
    return n % 2 == 1

L = list(filter(is_odd, range(1, 10)))

# lambda改造
new_l = list(filter(lambda x: x % 2 == 1, range(1, 10)))
```
# 总结
这一节，我们一起学习了Python中的匿名函数lambda，它的主要用途是减少代码的复杂度。需要注意的是lambda是一个表达式，并不是一个语句；它只能写成一行的表达形式，语法上并不支持多行。匿名函数通常的使用场景是：程序中需要使用一个函数完成一个简单的功能，并且该函数只调用一次。

**文中示例代码**： [python-learning](https://github.com/HamptonChen/python-learning)

未完待续，持续更新中......