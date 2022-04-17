---
title: LeetCode题解|206.反转链表
urlname: leetcode-reverse-linked-list
tags:
  - LeetCode题解
  - 面试
  - 链表
categories:
  - [算法, LeetCode]
abbrlink: 47373
date: 2020-12-17 23:50:48
---

## 前言
链表（Linked List）是一种常见的基础数据结构，是一种线性表，但是并不会按线性的顺序存储数据，而是在每一个节点里存到下一个节点的指针（Pointer）。

链表的结构如下图所示：

![](https://image.chenhanpeng.com/static/blog-images/blogImages/2020/20201217235816.png)

由于不是必须按顺序存储，链表在插入的时候可以达到`O(1)`的复杂度，但是在查找一个节点或者访问特定编号的节点则需要`O(n)`的时间。

对于如何反转单向链表、移除链表元素、合并两个链表等问题都是面试中比较常遇到的，今天我们来看LeetCode上的一道题：反转链表。

题目链接：https://leetcode-cn.com/problems/reverse-linked-list/


## 题目描述
反转一个单链表。

**示例 1:**
```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

该题主要有两种解法：循环实现和递归实现

## 解法1：循环实现
### 思路
1、初始化一个pre指针指向null，一个cur指针指向head。

2、开始遍历列表，在每一次循环中：
- 先保存cur.next;
- 把cur.next倒装方向指向pre;
- pre和cur都分别向前一步

### 图解
![](https://image.chenhanpeng.com/static/blog-images/blogImages/2020/20201218073444.png)

### 代码
```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var reverseList = function(head) {
    let pre = null;
    let cur = head;
    while (cur != null) {
        let tmp = cur.next;
        cur.next = pre;
        pre = cur;
        cur = tmp;
    }
    return pre;
};
```

## 解法2：递归实现
### 思路
我们可以将链表分为两部分：
- 第一个节点
- 余下的部分

假设余下的部分都是反转好的列表，那么我们只需要将已经反转好的这部分链表的最后一个节点指向原来的第一个节点，并且返回已经返回好的head。

对于上述说的余下部分，我们也可以将链表分为两部分：
- 第一个节点
- 余下的部分

由此，我们就可以用递归的方式来实现。递归的关键在于出口，在反转链表中，出口就是当链表被分成只剩下最后一个节点的时候，我们只需要直接返回当前节点作为 head。

### 图解
![](https://image.chenhanpeng.com/static/blog-images/blogImages/2020/20201218074412.png)

### 代码
```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var reverseList = function(head) {
    if (head == null || head.next == null) return head;
    let newHead = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
};
```