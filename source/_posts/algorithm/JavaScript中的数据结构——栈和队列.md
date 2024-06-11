---
title: JavaScript中的数据结构——栈和队列
urlname: javascript-stack-and-queue
tags:
  - 数据结构
  - 栈和队列
categories:
  - 算法
abbrlink: 44107
date: 2021-01-18 23:04:05
---

在前面 [JavaScript中的数据结构——链表](http://www.chenhanpeng.com/javascript-linked-list/) 一文中，我们学习了链表。今天我们一起来学习另外两种数据结构：栈和队列。

## 栈(Stack)
### 定义
栈是一种特殊的列表，限定仅在表尾进行插入和删除操作的线性表。表尾这一端我们称为栈顶，相对地，把另一端称为栈底。

栈遵循后进先出（LIFO）原则进行存储数据，先进入的数据被压入栈底，最后进入的数据在栈顶，需要读取数据的时候从栈顶开始弹出数据。如下图所示：

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210116204449.png)

因为栈的后进先出的特定，因此只能访问在栈顶的元素。

栈的操作主要有两种：入栈和出栈。

### 栈的实现

要实现一个栈，可以用数组或者链表来实现。下面我们用数组的方式来实现栈，代码如下：

```js
class Stack {
  constructor() {
    this.data = [];
  }

  // 入栈
  push(item) {
    this.data.push(item);
  }

  // 出栈
  pop() {
    // 如果栈为空，直接返回null
    if(this.data.length === 0) {
      return null;
    }
    return this.data.pop()
  }

  // 清空栈
  clear(){
    this.data = [];
  }
  get length(){
    return this.data.length;
  }
}
```


## 队列(Queue)
### 定义
队列和栈有点像，它们都是线性表，元素都是有序的。

但是和栈不同的是，队列遵循的是先进先出（FIFO）的原则，也就是从尾部添加元素，从头部移除元素，最新添加的元素必须排列在队列的末尾。

队列的主要操作有：往队列中插入新的元素和删除最早加入的元素。

### 队列的实现

队列同样可以用数组或链表来实现。下面我们还是用数组的方式来实现，代码如下：
```js
class Queue {
  constructor () {
    this.data = [];
  }
  // 入栈
  enqueue(item) {
    this.data.push(item)
  }
  // 出栈
  dequeue() {
    if (this.data.length === 0) {
      return null
    }
    return this.data.shift()
  }
  clear(){
    this.data = []
  }
  get length(){
    return this.data.length
  }
}
```


## 总结
栈遵循的是后进先出原则，队列遵循的是先进先出原则。

栈的队列实现起来还是相对比较简单，它们是用来帮我们组织数据的。比如下面两个实际开发场景：
- 使用堆栈来组织数据，实现文本编辑器的撤销操作；
- 使用队列处理数据，实现浏览器的事件循环处理事件。