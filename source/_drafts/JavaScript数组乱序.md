---
title: JavaScript数组乱序
urlname: javascript-array-shuffle
tags:
  - JavaScript
  - 乱序
categories:
  - 前端
  - JavaScript
---
## 前言

对数组进行排序对我们来说很容易就能够实现，但是你有考虑过如何对一个有序的数组实现乱序，即随机排序吗？

数组乱序在实际开发过程中是可能碰到的，下面我们一起看看如何实现数组乱序。

## sort + Math.random
我们一开始可能会想到利用数组的sort方法，判断随机出来的0-1的值与0.5的大小，实现排序。该方法实现如下：

```javascript
var arr = [1, 2, 3, 4, 5, 6];

arr.sort(function(){
    return Math.random() - 0.5;
});

console.log(arr)
```

上面的实现方法看起来很完美地实现了乱序的需求，但实际的效果如何我们还是要进行测试。

## 对sort + Math.random方法进行测试
将[1, 2, 3, 4, 5, 6]数组进行乱序10万次，计算乱序后每个值出现在各个位置的概率。测试代码如下：

```javascript
const fisherYates = () => {
    const array = [0, 1, 2, 3];
    for (let i = 1; i < array.length; i++) {
        const random = Math.floor(Math.random() * (i + 1));
        [array[i], array[random]] = [array[random], array[i]];
    }
    return array;
};

const mathRandom = () => {
    const array = [0, 1, 2, 3];
    array.sort(() => Math.random() > .5);
    return array;
};

const TestCount = func => {
    const times = 10000000;
    const count = new Array(4).fill(0).map(() => new Array(4).fill(0));
    for (let i = 0; i < times; i++) {
        func().forEach((n, i) => count[n][i]++);
    }
    console.table(count.map(n => n.map(n => (n / times * 100).toFixed(2) + '%')));
};

TestCount(mathRandom)
```
