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
在Chrome浏览器上将[1, 2, 3, 4, 5]数组进行乱序10万次，计算乱序后每个值出现在各个位置的概率。测试代码如下：

```javascript
let statistics = new Array(5).fill(0).map(() => new Array(5).fill(0));
for (let i = 0; i < 100000; i++) {
	let arr = [1, 2, 3, 4, 5];
	arr.sort(() => Math.random() - 0.5);
	arr.forEach((value, index) => {
		statistics[index][value - 1]++;
	})
}
console.table(statistics.map(item => item.map(value => (value / 100000 * 100).toFixed(2) + '%')));
```

在Chrome浏览器上执行得到如下结果：

{% qnimg javascript/array_shuffle_1.png %}

从上图我们可以看出：元素出现在每个位置上的概率并没有趋向于相等，反而元素停留在原位置上的概率更大，所以Array.sort()方法进行乱序是存在问题的。要知道问题具体出现在哪，我们需要了解sort方法实现原理。

## Array.sort()方法的实现
ECMAScript对 Array.prototype.sort 的定义只规定了效果，没有要求用什么样的排序方式实现sort()方法，也没有要求是否要采用稳定排序算法，所以不同的浏览器实现的方式都不太一样。

### Chrome的sort
基于V8引擎，它的排序算法进行了很多的优化，但核心是小于等于10的数组用插入排序，大于10的采用快速排序。

### FireFox的sort
基于SpiderMonkey引擎，采用归并排序。

### Safari的sort
基于Nitro(JavaScriptCore) 引擎，如果没有自定义的排序规则传入，采用桶排序；传入自定义规则，采用归并排序。

### Microsoft Edge/IE9+ 的sort
基于Chakra引擎，采用快速排序。

### 分析
其实不管用什么排序方法，大多数排序算法的时间复杂度介于O(n)到O(n2)之间，元素之间的比较次数通常情况下要远小于n(n-1)/2，也就意味着有一些元素之间根本就没机会相比较（也就没有了随机交换的可能），这些 sort 随机排序的算法自然也不能真正随机。

怎么理解上边这句话呢？其实我们想使用array.sort进行乱序，理想的方案或者说纯乱序的方案是数组中每两个元素都要进行比较，这个比较有50%的交换位置概率。这样一来，总共比较次数一定为n(n-1)。而在sort排序算法中，大多数情况都不会满足这样的条件。因而当然不是完全随机的结果了。

这里我们以Chrome使用的插入排序算法为例：在插入排序的算法中，当待排序元素跟有序元素进行比较时，一旦确定了位置，就不会再跟位置前面的有序元素进行比较，所以就出现了前面说的“没有了随机交换的可能”的情况，导致乱序不彻底。

要实现真正意义上的乱序，这就要提到经典的 [洗牌算法Fisher–Yates_shuffle](https://en.wikipedia.org/wiki/Fisher–Yates_shuffle)。

## 洗牌算法Fisher–Yates_shuffle
Fisher–Yates_shuffle的原理如下：

> 1、写下从 1 到 N 的数字
2、取一个从 1 到剩下的数字（包括这个数字）的随机数 k
3、从低位开始，得到第 k 个数字（这个数字还没有被取出），把它写在独立的一个列表的最后一位
4、重复第 2 步，直到所有的数字都被取出
5、第 3 步写出的这个序列，现在就是原始数字的随机排列

简单的说：就是随机抽一个放到最后。把剩余的数继续抽，继续放到次后。。。。依次执行

实现代码如下：

```javascript
function shuffle(array) {
	var j, x, i;
	for (i = array.length; i; i--) {
		j = Math.floor(Math.random() * i);
		x = array[i - 1];
		array[i - 1] = array[j];
		array[j] = x;
	}
	return array;
}
```

我们对上面实现的方法进行10万次乱序：
```javascript
var statistics = new Array(5).fill(0).map(() => new Array(5).fill(0));
for (let i = 0; i < 100000; i++) {
	let arr = [1, 2, 3, 4, 5];
	arr = shuffle(arr);
	arr.forEach((value, index) => {
		statistics[index][value - 1]++;
	})
}
console.table(statistics.map(item => item.map(value => (value / 100000 * 100).toFixed(2) + '%')));
```

统计结果如下：
{% qnimg javascript/array_shuffle_2.png %}

从上图可以看出，数组中任意一个元素出现在任何一个位置的概率都是相等的，这样我们就真正实现了乱序的效果。
