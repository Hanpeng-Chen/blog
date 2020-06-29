---
title: JavaScript数组去重
tags:
  - JavaScript
  - 数组去重
categories:
  - 前端
  - JavaScript
abbrlink: 15019
date: 2020-06-14 22:57:57
urlname: javascript-array-unique
---
JavaScript的数组去重是前端比较常见的一个问题，今天我们来学习如何实现数组去重。


> 欢迎关注我的微信公众号：前端极客技术(FrontGeek)

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

## indexOf
针对上面的方法，我们可以利用 `indexOf` 来简化内层循环，将双层循环变为单层循环。实现代码如下：

```js
var array = [1, 1, '1', '1', 2, 3, '2']

function unique(arr) {
  var res = []
  for (var i = 0; i < arr.length; i++) {
    if (res.indexOf(arr[i]) === -1) {
      res.push(arr[i])
    }
  }
  return res
}

console.log(unique(array))   // [1, "1", 2, 3, "2"]
```

## sort()排序后相邻比对去重
如果我们先将数组进行排序，排序后相同的元素就会被排在一起，这样我们只要将当前元素和上一个元素进行比较，看两者是否相同，相同则说明重复，不相同push进res中。

实现代码如下：
```js
var array = [1, 1, '1', '1', 2, 3, '2']

function unique(arr) {
  var res = []
  var sortedArr = arr.concat().sort()
  var pre
  for (var i = 0; i < sortedArr.length; i++) {
    if (!pre || pre !== sortedArr[i]) {
      res.push(sortedArr[i])
    }
    pre = sortedArr[i]
  }
  return res
}

console.log(unique(array))   // [1, "1", 2, "2", 3]
```

## filter
ES5 提供了filter方法，我们可以用来简化外层循环。indexOf和排序后去重分别使用filter方法的实现代码如下：

```js
var array = [1, 1, '1', '1', 2, 3, '2']

// indexOf
function unique(arr) {
  var res = arr.filter(function(item, index, arr) {
    return arr.indexOf(item) === index
  })
  return res
}
console.log(unique(array))

// sort
function sortedUnique(arr) {
  return arr.concat().sort().filter(function(item, index, arr) {
    return !index || item !== arr[index - 1]
  })
}
```

## Set()和Map()
如果你熟悉ES6的话，就知道利用Set()和Map()两个数据结构，我们也可以实现数组去重。

我们先来看下Set()的实现代码：

```javascript
var array = [1, 1, '1', '1', 2, 3, '2']

function unique(arr) {
  return Array.from(new Set(arr))
}
console.log(unique(array))

// 另一种写法
function unique(arr) {
  return [...new Set(arr)]
}
```

接下来我们来看下Map()的实现代码：
```js
var array = [1, 1, '1', '1', 2, 3, '2']

function unique(arr) {
  const tmp = new Map()
  return arr.filter(item => !tmp.has(item) && tmp.set(item, 1))
}
```

## 特殊类型比较
上面我们去重的数组包含的元素都是简单的string和number，但是数组的元素类型还可以是null、undefined、NaN、对象等，针对这些元素，用我们上面实现的方法是有问题的。

假设我们有下面这样一个数组需要去重，并且用上面的提到的方法去重，结果如下：
```js
var array = ['true','true',true,true,0,0,1,1,'1','1',15,15,false,false,undefined,undefined,null,null,NaN,NaN,'NaN','NaN','a','a',{},{},{a:2},{a:2}];

// 去重结果如下：
// 双层循环
["true", true, 0, 1, "1", 15, false, undefined, null, NaN, NaN, "NaN", "a", {}, {}, {a:2}, {a:2}]

// indexOf
["true", true, 0, 1, "1", 15, false, undefined, null, NaN, NaN, "NaN", "a", {}, {}, {a:2}, {a:2}]

// filter + indexOf
["true", true, 0, 1, "1", 15, false, undefined, null, "NaN", "a", {}, {}, {a:2}, {a:2}]

// filter + sort
 [0, 1, "1", 15, NaN, NaN, "NaN", {}, {}, {a:2}, {a:2}, "a", false, null, "true", true, undefined]

// Set
["true", true, 0, 1, "1", 15, false, undefined, null, NaN, "NaN", "a", {}, {}, {a:2}, {a:2}]

// Map
["true", true, 0, 1, "1", 15, false, undefined, null, NaN, "NaN", "a", {}, {}, {a:2}, {a:2}]
```

将各种方法和去重结果整理成下面的表格：

|  方法    |  去重结果    |
| ---- | ---- |
|   双层循环 ===判断   |  对象和NaN没去重 |
|  indexOf |   对象和NaN没去重   |
|  filter + indexOf  |   对象没去重，NaN被忽略掉了   |
| filter + sort  |   对象和NaN没去重   |
|   Set  |  对象没去重    |
|  Map  |   对象没去重   |


通过上面的表格，我们可以看出主要是对象和NaN两种类型的元素需要注意。

使用=== 判断两个NaN是否相等，得到的结果为false。

## 优化
如果元素是对象，我们利用Object键值对的方式来进行去重，利用JSON.stringify将对象序列化后存为Object的key值，比如：`Object[value]=1`，在判断另一个元素时，如果Object[value2]存在的话，说明该值重复。

如果元素不是对象，我们使用includes来判断新的数组res中是否有当前值。

实现代码如下：

```javascript
var array = ['true','true',true,true,0,0,1,1,'1','1',15,15,false,false,undefined,undefined,null,null,NaN,NaN,'NaN','NaN','a','a',{},{},{a:2},{a:2}];

function unique(arr) {
  var res = []
  var obj = {}
  arr.forEach(item => {
    if (typeof item !== 'object') {
      if (!res.includes(item)) {
        res.push(item)
      }
    } else {
      var valueStr = JSON.stringify(item)
      if (!obj[valueStr]) {
        res.push(item)
        obj[valueStr] = 1
      }
    }
  })
  return res
}
console.log(unique(array))
// ["true", true, 0, 1, "1", 15, false, undefined, null, NaN, "NaN", "a", {}, {a:2}]
```