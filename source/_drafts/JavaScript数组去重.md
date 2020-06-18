---
title: JavaScript数组去重
tags: javascript-array-unique - JavaScript - 数组去重
categories:
  - 前端
  - JavaScript
abbrlink: 15019
date: 2020-06-14 22:57:57
urlname:
---
JavaScript的数组去重是前端面试中比较常见的一道题，今天我们一起来看看这道题。

## 双层循环
大部分人最先想到的是通过两次循环解决，我们先新建一个空的数组res，通过遍历待去重数组array和res，判断array[i]是否存在于res中，如果不存在，将array[i]push到res中。

我们直接看下面的代码：
```javascript
var array = [1, 1, '1', '1', 2, 3, '2']

function unique(arr) {
  var res = []
  for (var i = 0; i < arr.length; i++) {
    for (var j = 0, resLength = res.length; j < resLength; j++) {
      if (arr[i] === res[j]) {
        break
      }
    }
    if (j === resLength) {
      res.push(arr[i])
    }
  }
  return res
}

console.log(unique(array))   // [1, "1", 2, 3, "2"]
```