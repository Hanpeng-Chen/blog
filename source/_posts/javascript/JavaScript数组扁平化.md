---
title: JavaScript-数组扁平化
urlname: javascript-array-flatten
tags:
  - JavaScript
  - 数组扁平化
categories:
  - [前端, JavaScript]
abbrlink: 46152
date: 2020-06-20 16:28:25
---

上一篇文章我们将JavaScript中数组的方法汇总了一下，本文我们一起来看看JavaScript数组常见的一个问题：数组扁平化。

## 什么是数组扁平化
数组扁平化：就是讲一个复杂的嵌套多层的数组，一层一层地转化为层级较少或者只有一层的数组。

下面我们通过实际例子来看看都有哪些解决方法：
```js
let array = [1, [2, [3, [4, 5]]]]
// 需要将上面的array展开得到下面的一维数组
[1, 2, 3, 4, 5]
```

> 欢迎关注我的微信公众号：前端极客技术(FrontGeek)

## flat()
从上一篇文章中我们可以知道，在ES6 新增的flat()方法可以用于将嵌套的数组“拉平”，变成一维的数组。

```js
let array = [1, [2, [3, [4, 5]]]]
array.flat()  // [1, 2, [3, [4, 5]]]
```

但是flat()方法默认只会“拉平”一层，如果不管多少层嵌套，都要转成一维数组，可以用`Infinity`关键字作为参数。
```js
let array = [1, [2, [3, [4, 5]]]]
array.flat(Infinity)  // [1, 2, 3, 4, 5]
```

## replace + JSON.parse
除了直接使用上面说的flat()方法，我们还可以通过replace和JSON.parse方法结合将数组扁平化。

我们先利用JSON.stringify将数组转出JSON字符串，在通过replace方法，利用正则表达式将字符串中的`[`、`]`替换成掉，再在字符串首尾加上[]，最后利用JSON.parse方法将字符串重新转为数组。

```js
let array = [1, [2, [3, [4, 5]]]]
let str = JSON.stringify(array)  // "[1,[2,[3,[4,5]]]]"
str = str.replace(/(\[|\])/g, '')  // "1,2,3,4,5"
str = '[' + str + ']'  // "[1,2,3,4,5]"
let arr = JSON.parse(str) // [1, 2, 3, 4, 5]
```

## 递归
除了上面两种方法，我们比较容易想到的是循环数组元素，如果元素还是一个数组，就递归调用该方法。实现代码如下：

```js
let array = [1, [2, [3, [4, 5]]]]

function flatten(arr) {
    let result = []
    for (let i = 0; i < arr.length; i++) {
        if (Array.isArray(arr[i])) {
            result = result.concat(flatten(arr[i]))
        } else {
            result.push(arr[i])
        }
    }
    return result
}

console.log(flatten(array))  // [1, 2, 3, 4, 5]
```

## reduce函数迭代
从上面递归的方法来看，其实就是对数组进行处理，最终返回一个一维数组，那么我们可以考虑使用 `reduce` 来简化代码：

```js
let array = [1, [2, [3, [4, 5]]]]

function flatten(arr) {
    return arr.reduce(function (prev, next) {
        return prev.concat(Array.isArray(next) ? flatten(next) : next)
    }, [])
}

console.log(flatten(array))  // [1, 2, 3, 4, 5]
```

## 扩展运算符...
ES6 增加了扩展运算符，用于取出参数对象的所有可遍历属性，拷贝到当前对象中。

我们先通过下面的例子来看看扩展运算符的用法：
```js
let array = [1, [2, [3, [4, 5]]]]

console.log([].concat(...array))  // [1, 2, [3, [4, 5]]]
```

我们可以使用扩展运算符对递归方法进行修改，具体代码如下：
```js
let array = [1, [2, [3, [4, 5]]]]

function flatten(arr) {
    while (arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr)
    }
    return arr
}

console.log(flatten(array))  // [1, 2, 3, 4, 5]
```