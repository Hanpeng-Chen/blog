---
title: Python从小白到攻城狮(12)——错误和异常
date: 2019-09-27 10:12:37
tags:
  - python教程
  - 错误/异常
categories:
  - Python
---

和其他语言一样，异常处理是Python中一种很常见，并且很重要的机制和代码规范。

Python有两种错误很容易辨认：**语法错误**和**异常**。

# 语法错误
**语法错误**: 就是你写的代码不符合编程规范，无法被识别与执行。比如下面这个例子：
```python
if name is not None
    print(name)
```
if语句漏了冒号，不符合Python的语法规范，所以程序报错 `invalid syntax`
```
  File "invalid_syntax.py", line 2
    if name is not None
                      ^
SyntaxError: invalid syntax
```


# 异常
**异常**：程序的语法正确，也可以被执行，但在运行期检测到的错误被称为异常。

大多数的异常都不会被程序处理，都以错误信息的形式展现在这里：
```python
>>> 10 / 0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
>>> 4 + num * 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'num' is not defined
>>> 1 + [1, 3]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'list'
```
异常以不同的类型出现，这些类型都作为信息的一部分打印出来，例子中的类型有：ZeroDivisionError，NameError，TypeError。


错误信息的前面部分显示了异常发生的上下文，并以调用栈的形式显示具体信息。

当然，Python中还有很多其他异常类型，比如`KeyError`是指字典中的键找不到；`FileNotFoundError`是指发送了读取文件的请求，但相应的文件不存在等等，我在此不一一赘述，你可以自行参考[相应文档](https://docs.python.org/3/library/exceptions.html#bltin-exceptions)。

# 异常处理
如果执行到程序中某处抛出了异常，程序就会被终止并退出。那有没有什么办法可以不终止程序，让其照样运行下去呢？答案当然是肯定的，这也就是我们所说的异常处理，通常使用**try**和**except**来解决。

try语句按照如下方式工作：
* 首先，执行try子句（在关键字try和关键字except之间的语句）
* 如果没有异常发生，忽略except子句，try子句执行后结束
* 如果在执行try子句的过程中发生了异常，那么try子句余下的部分将被忽略。如果异常的类型和 except 之后的名称相符，那么对应的except子句将被执行。最后执行 try 语句之后的代码。
* 如果一个异常没有与任何的except匹配，那么这个异常将会传递给上层的try中。


处理程序将只针对对应的try子句中的异常进行处理，而不是其他的 try 的处理程序中的异常。

一个except子句可以同时处理多个异常，这些异常将被放在一个括号里成为一个元组，例如:
```python
try:
    s = input('please enter two numbers separated by comma: ')
    num1 = int(s.split(',')[0].strip())
    num2 = int(s.split(',')[1].strip())
    ...
except (ValueError, IndexError) as err:
    print('Error: {}'.format(err))
    
print('continue')
```

一个 try 语句可能包含多个except子句，分别来处理不同的特定的异常。最多只有一个分支会被执行。
```python
try:
    s = input('please enter two numbers separated by comma: ')
    num1 = int(s.split(',')[0].strip())
    num2 = int(s.split(',')[1].strip())
    ...
except ValueError as err:
    print('Value Error: {}'.format(err))
except IndexError as err:
    print('Index Error: {}'.format(err))

print('continue')
```

不过很多时候，我们很难保证程序覆盖所有的异常类型，通常的做法是在最后一个except block，声明其处理的异常类型是**Exception**。Exception是其他所有非系统异常的基类，能够匹配任意非系统异常。代码如下：
```python
try:
    s = input('please enter two numbers separated by comma: ')
    num1 = int(s.split(',')[0].strip())
    num2 = int(s.split(',')[1].strip())
    ...
except ValueError as err:
    print('Value Error: {}'.format(err))
except IndexError as err:
    print('Index Error: {}'.format(err))
except Exception as err:
    print('Other Error: {}'.format(err))

print('continue')
```

最后一个except子句可以忽略异常的名称，它将被当做通配符使用，表示与任意异常相匹配（包括系统异常等）。
```python
try:
    s = input('please enter two numbers separated by comma: ')
    num1 = int(s.split(',')[0].strip())
    num2 = int(s.split(',')[1].strip())
    ...
except ValueError as err:
    print('Value Error: {}'.format(err))
except IndexError as err:
    print('Index Error: {}'.format(err))
except:
    print('Other error')

print('continue')
```


异常处理中，还有一个很常见的用法是**finally**，经常和try、except放在一起用。无论发生什么情况，finally block中的语句都会被执行，哪怕前面的try和except block中使用了return语句。

常见的应用场景如：读取文件：
```python
import sys
try:
    f = open('file.txt', 'r')
    .... # some data processing
except OSError as err:
    print('OS error: {}'.format(err))
except:
    print('Unexpected error:', sys.exc_info()[0])
finally:
    f.close()
```

try except 语句还有一个可选的**else**子句，如果使用这个子句，那么必须放在所有except子句之后。这个子句将在try子句没有发生任何异常的时候执行。例如：

```python
try:
    s = input('please enter two numbers separated by ",":')
    num1 = int(s.split(',')[0].strip())
    num2 = int(s.split(',')[1].strip())
except ValueError as err:
    print('Value Error: {}'.format(err))
else:
    print('in else')
```
使用 else 子句比把所有的语句都放在 try 子句里面要好，这样可以避免一些意想不到的、而except又没有捕获的异常。


异常处理并不仅仅处理那些直接发生在try子句中的异常，而且还能处理子句中调用的函数（甚至间接调用的函数）里抛出的异常。例如:
```python
def this_fails():
    x = 1/0
   
try:
    this_fails()
except ZeroDivisionError as err:
    print('Handling run-time error:', err)

#######输出打印
Handling run-time error: division by zero
```


# 抛出异常
Python使用**raise**语句抛出一个指定的异常。例如：
```python
>>> raise NameError('Error')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: Error
```

raise唯一的一个参数指定了要被抛出的异常。它必须是一个异常的实例或者是异常的类。

如果你只想知道这是否抛出了一个异常，并不想去处理它，那么一个简单的 raise 语句就可以再次把它抛出。

```python
try:
    raise NameError('HiThere')
except NameError:
    print('An exception flew by!')
    raise

###输出
An exception flew by!
Traceback (most recent call last):
  File "deal_error.py", line 23, in <module>
    raise NameError('HiThere')
NameError: HiThere
```

# 用户自定义异常
我们可以通过创建一个新的异常类来拥有自己的异常。异常类继承自Exception类，可以直接继承、或者间接继承，例如：

```python
class MyInputError(Exception):
    """Exception raised when there're errors in input"""
    def __init__(self, value): # 自定义异常类的初始化
        self.value = value
    def __str__(self): # 自定义异常类的String表达形式
        return ("{} is invalid input".format(repr(self.value)))

try:
    raise MyInputError(1)
except MyInputError as error:
    print('error: {}'.format(error))
```
执行上面代码块，得到下面的结果：
```
error: 1 is invalid input
```

实际工作中，如果内置的异常类型无法满足我们的需求，或者为了让异常更加详细、可读，想增加一些异常类型的其他功能，我们可以自定义所需异常类型。不多大多数情况下，Python内置的异常类型就足够我们是使用了。


# 总结

- 异常，通常是指程序运行的过程中遇到了错误，终止并退出。我们通常使用try except语句去处理异常，这样程序就不会被终止，仍能继续执行。
- 处理异常时，如果有必须执行的语句，比如文件打开后必须关闭等等，则可以放在finally block中。
- 异常处理，通常用在你不确定某段代码能否成功执行，也无法轻易判断的情况下，比如数据库的连接、读取等等。正常的flow-control逻辑，不要使用异常处理，直接用条件语句解决就可以了。


**文中示例代码**： [python-learning](https://github.com/HamptonChen/python-learning)

未完待续，持续更新中......