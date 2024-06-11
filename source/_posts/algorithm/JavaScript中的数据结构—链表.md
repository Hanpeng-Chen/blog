---
title: JavaScript中的数据结构—链表
urlname: javascript-linked-list
tags:
  - 数据结构
  - 链表
categories:
  - 算法
abbrlink: 31834
date: 2021-01-17 12:17:31
---
## 前言
数据结构与算法在前端开发工程师的日常工作中也许不常用，但在这对前端工程师要求日益提高的时代，如果对数据结构、算法思维、代码效率等知识拥有足够的储备，那么我们将拥有更强的竞争力。

话不多说，我们接下来学习一种数据结构：链表(Linked list)。

## 链表
数组对于每个开发来说是非常熟悉的一种数据结构。链表是一种比数组稍微复杂一点的数据结构，掌握起来也比数组稍难一些。

链表是一种与数组类似的线性数据结构，但与数组的元素存储在特定的内存位置或索引中不同，链表的每个元素都是独立的对象，它包含一个指向该列表中下一个对象的指针或链接。

链表的每个元素（我们通常称为节点）包含两项：
- 存储的数据：数据可以是任何有效的数据类型。
- 指针：到下一节点的链接


链表的结构可以由很多种，它可以是单链表或双链表，也可以是已排序的或未排序的，环形的或非环形的。如果一个链表是单向的，那么链表中的每个元素没有指向前一个元素的指针。已排序的和未排序的链表较好理解。常见的有：单向链表、双向链表、单向循环链表和双向循环链表。

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210114213201.png)

从上图我们可以看出，循环链表和单链表的区别在于：单链表的尾元素指向的Null，而循环链表的尾元素指向的是链表的头部元素。

> 欢迎关注我的微信公众号：**前端极客技术(FrontGeek)**

## 单向链表

链表和数组一样，也支持数据的查找、插入和删除。

由于链表是非连续的，想要访问第i个元素就不像数组那么方便，而是需要根据指针一个节点一个节点一次遍历，直到找到相应的节点。

因为链表的数据本身是非连续的空间，所以它的插入或删除操作，不需要像数组那边挪动原来的数据，因此在链表中插入数据、删除数据是非常快的。

### 设计链表
我们设计链表包含两个类：一个是Node类用来表示节点；另一个是LinkedList类提供插入节点、删除节点等操作。

头结点head的next初始化为null，每当调用插入方法时，next就会指向新的元素

- Node
```js
class Node {
  constructor(el) {
    this.el = el;
    this.next = null;
  }
}
```

- LinkedList
```js
class LinkedList {
  constructor(){
    this.head = new Node('head')
  }

  // 用于查找
  find(){}

  // 插入节点
  insert(){}

  // 删除节点
  remove(){}
}
```

### 查找
从头结点开始查找，如果没找到就把当前指针往后移，找到就返回该元素，如果遍历完没找到就直接返回null。

代码实现如下：
```js
  find(item){
    let currentNode = this.head
    while (currentNode !== null && currentNode.el !== item) {
      currentNode = currentNode.next;
    }
    return currentNode
  }
```

### 插入新节点

要往链表中插入新节点，需要知道在哪个节点后面插入。那么我们就需要知道在哪里插入和插入的元素是什么。

知道在哪个节点后面插入后，首先我们要先找到这个节点的位置，这里我们就可以用上面实现的查找方法。

找到节点后，我们先创建一个新节点，把新节点的next指针指向找到的这个节点next指向的对应节点，再把找到的这个节点的next指针指向新节点，数据的插入就完成了。具体过程如下图所示：

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210114215737.png)

代码实现如下：
```js
// 插入节点
  // el:要插入的数据
  // item：数据插入到这个节点后面
  insert(el, item){
    const newNode = new Node(el)
    const cur = this.find(item)
    newNode.next = cur.next
    cur.next = newNode
  }
```


### 删除节点
删除节点和插入节点类似，首先要找到相应节点的前一个节点，找到后，让它的next指向待删除节点的下一个节点。如下图所示：

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210114220545.png)

代码实现如下：
```js
  findPre(item) {
    let cur = this.head
    while (cur.next !== null && cur.next.el !== item) {
      cur = cur.next
    }
    return cur
  }

  // 删除节点
  remove(item){
    const preNode = this.findPre(item)
    if (preNode.next !== null) {
      preNode.next = preNode.next.next
    }
  }
```

单向链表完整代码：
```js
// 单项链表

// 节点
class Node {
  constructor(el) {
    this.el = el;
    this.next = null;
  }
}

class LinkedList {
  constructor(){
    this.head = new Node('head')
  }

  // 用于查找
  find(item){
    let currentNode = this.head
    while (currentNode !== null && currentNode.el !== item) {
      currentNode = currentNode.next;
    }
    return currentNode
  }

  findPre(item) {
    let cur = this.head
    while (cur.next !== null && cur.next.el !== item) {
      cur = cur.next
    }
    return cur
  }

  // 插入节点
  // el:要插入的数据
  // item：数据插入到这个节点后面
  insert(el, item){
    const newNode = new Node(el)
    const cur = this.find(item)
    newNode.next = cur.next
    cur.next = newNode
  }

  // 删除节点
  remove(item){
    const preNode = this.findPre(item)
    if (preNode.next !== null) {
      preNode.next = preNode.next.next
    }
  }
}
```




## 双向链表
双向链表，顾名思义就是它有两个方向，除了next指针指向下一个节点外，比单向链表多了一个prev指针，用来指向上一个节点。

双向链表如下图所示：

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210114223754.png)

和单向链表相比，在存储相同的数据情况下，双向链表要比单向链表占用更多的空间，但双向链表往往会比单向链表更加灵活。例如：

双向链表删除节点时，因为有prev指向上一个节点，就不需要像单向链表一样去寻找待删除节点的前驱节点，使得删除节点的效率提高了。


接下来我们来看下如何实现双向链表：

### 双向链表的设计
对于节点类，我们只要在单向链表的基础上，加上一个prev指针。

```js
class Node {
  constructor(el) {
    this.el = el
    this.next = null
    this.prev = null
  }
}
```

另外双向链表的查找和单向链表一样，这里就不再细说，我们直接来看下插入节点。
### 插入
双向链表的插入和单向链表相似，多了一个prev指针，只要将新节点的prev指向前驱节点，将后驱节点的prev指向新节点。如下图所示：

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210114224936.png)

实现代码如下：
```js
insert(el, item){
    const newNode = new Node(el)
    const cur = this.find(item)
    newNode.next = cur.next
    newNode.prev = cur
    cur.next = newNode
    cur.next.prev = newNode
  }
```

### 删除
双向链表的删除 remove 方法比单链表效率高，不需要查找前驱节点，只要找出待删除节点，然后将该节点的前驱 next 属性指向待删除节点的后继，设置该节点后继 previous 属性，指向待删除节点的前驱即可。

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210114225122.png)

实现代码如下：
```js
remove(item){
    const node = this.find(item)
    node.prev.next = node.next
    node.next.prev = node.prev
    node.next = null
    node.prev = null
  }
```

双向链表完整代码如下：
```js
// 双向链表
class Node {
  constructor(el) {
    this.el = el
    this.next = null
    this.prev = null
  }
}

class LinkedList {
  constructor(){
    this.head = new Node('head')
  }

  // 用于查找
  find(item){
    let currentNode = this.head
    while (currentNode !== null && currentNode.el !== item) {
      currentNode = currentNode.next;
    }
    return currentNode
  }

  // 插入节点
  // el:要插入的数据
  // item：数据插入到这个节点后面
  insert(el, item){
    const newNode = new Node(el)
    const cur = this.find(item)
    newNode.next = cur.next
    newNode.prev = cur
    cur.next = newNode
    cur.next.prev = newNode
  }

  // 删除节点
  remove(item){
    const node = this.find(item)
    node.prev.next = node.next
    node.next.prev = node.prev
    node.next = null
    node.prev = null
  }
}
```

## 单向循环链表
单向循环链表和单向链表相似，节点类型都一样，唯一的区别就是在创建循环链表的时候，让其头结点的next属性指向它本身。
```js
head.next = head
```

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210114230407.png)

单向循环链表如上图所示，具体的实现细节不再一一细说，我们直接来看实现代码：
```js
class Node {
  constructor(el) {
    this.el = el;
    this.next = null;
  }
}

class LinkedList {
  constructor(){
    this.head = new Node('head')
    this.head.next = this.head
  }

  // 用于查找
  find(item){
    let currentNode = this.head
    while (currentNode !== null && currentNode.el !== item) {
      currentNode = currentNode.next;
    }
    return currentNode
  }

  findPre(item) {
    let cur = this.head
    while (cur.next !== null && cur.next.el !== item) {
      cur = cur.next
    }
    return cur
  }

  // 插入节点
  // el:要插入的数据
  // item：数据插入到这个节点后面
  insert(el, item){
    const newNode = new Node(el)
    const cur = this.find(item)
    newNode.next = cur.next
    cur.next = newNode
  }

  // 删除节点
  remove(item){
    const preNode = this.findPre(item)
    if (preNode.next !== null) {
      preNode.next = preNode.next.next
    }
  }
}
```


## 双向循环链表
既然单链表可以有循环链表，双向链表也可以是循环链表。双向循环链表头结点的prev指针指向尾结点，尾结点的next指针指向头结点。如下图所示：

![](https://pub-9effe6ef78a64cfc92922e0f4e06f7dd.r2.dev/blog-images/blogImages/2021/20210114231724.png)

实现代码如下：
```js

```

本文所有示例代码见：[linked-list]()

## 参考文章：
- [JavaScript数据结构之链表--介绍](https://juejin.cn/post/6844903798754770958)
- [JS中的算法与数据结构——链表(Linked-list)](https://juejin.cn/post/6844903498362912775)
