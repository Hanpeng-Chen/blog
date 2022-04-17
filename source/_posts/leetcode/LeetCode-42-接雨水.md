---
title: 「LeetCode系列」42.接雨水
tags:
  - LeetCode题解
  - 面试
categories:
  - [算法, LeetCode]
abbrlink: 56686
date: 2021-03-05 10:37:45
urlname:
---

> 作者：Hanpeng_Chen
>
> 公众号：**前端极客技术**
>
> 博客：[官网](http://www.chenhanpeng.com)、[掘金](https://juejin.cn/user/1063982988795911/posts)



今天我们来做一道LeetCode上的题目，原题链接：[42.接雨水](https://leetcode-cn.com/problems/trapping-rain-water)

## 题目描述
给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

**示例1：**
![](https://image.chenhanpeng.com/static/blog-images/blogImages/2021/20210302213550.png)

> 输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
>
> 输出：6
>
> 解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。

**示例2：**

> 输入：height = [4,2,0,3,2,5]
>
> 输出：9


## 思路分析

### 暴力求解
**思路**

根据示例1的图片我们可以比较清晰理解题目：每个柱子宽度为1，所以我们只要计算出每个柱子上面可以积蓄雨水的体积，最后进行求和即可得出总的雨水体积。

对于每个柱子上面的蓄水量，根据木桶原理，我们只要找出下雨后水能达到的最高位置，即两边最大高度的较小值减去当前高度的值。

**求解步骤：**
- 初始化sum = 0
- 从左往右遍历数组：
  - 初始化两边最大高度分别为0： left_max = 0, right_max = 0
  - 从当前元素向左遍历并更新：
    - left_max = Math.max(left_max, height[k])
  - 从当前元素向右遍历并更新：
    - right_max = Math.max(right_max, height[k])
  - 将Math.min(left_max, right_max) - height[i]累加到sum中


**实现代码：**
```js
var trap = function(height) {
    let len = height.length;
    let sum = 0;
    for (let i = 1; i < len - 1; i++) {
        let left_max = 0, right_max = 0;
        for (let k = i; k >= 0; k--) {
            left_max = Math.max(left_max, height[k])
        }
        for (let k = i; k < len; k++) {
            right_max = Math.max(right_max, height[k])
        }
        sum += (Math.min(left_max, right_max) - height[i])
    }
    return sum
};
```

**复杂度分析**

该方法用到了双重遍历，所以时间复杂度为：O(n^2)；空间复杂度为：O(1)


### 暴力求解优化
上面的暴力解法中，每次为了找到位置i的左右最大值，都要向左和向右遍历一次，因此出现了双重遍历。如果我们提前将每个位置的左右最大值查出并存储起来，后面遍历数组的时候直接去相应的值，那么就不会出现双重遍历的情况，时间复杂度也就降为：O(n)。

**步骤**
- 找到数组中i的到最左端最高的柱子高度 left_max;
- 找到数组中i到最右端最高的柱子的高度right_max;
- 遍历数组：
    - 累加  Math.min(left_max[i], right_max[i]) - height[i]


**实现代码**
```js
var trap = function(height) {
    if (!height || height.length === 0) return 0;
    let sum = 0;
    let left_max = [];
    let right_max = [];
    let len = height.length;
    left_max[0] = height[0];
    for (let i = 1; i < len; i++) {
        left_max[i] = Math.max(height[i], left_max[i - 1])
    }
    right_max[len - 1] = height[len - 1];
    for (let i = len - 2; i >= 0; i--) {
        right_max[i] = Math.max(height[i], right_max[i + 1])
    }
    for (let i = 1; i < len - 1; i++) {
        sum += Math.min(left_max[i], right_max[i]) - height[i];
    }
    return sum;
};
```

**复杂度分析**
- 时间复杂度：总共用了三次遍历，每次O(n)，所以总的时间复杂度为O(n)
- 空间复杂度：使用了额外的O(n)空间来存left_max和right_max数组，所以空间复杂度为O(n)

该解法是通过空间换时间的方式来减少运行时间。

### 双指针
接下来我来介绍另一种比较好的解法：双指针解法。

在暴力解法的核心主要是查找出左右两边的最大值，但是我们仔细分析会发现，只要 right_max[i] > left_max[i]，柱子i位置的最大存水量就由left_max决定；反之如果 left_max[i] > right_max[i]，则由right_max决定。

根据这个思路，我们还是先从左往右扫描数组，我们可以认为如果一侧有更高的柱子（例如右侧），积水的高度依赖于当前方向的高度（从左到右）。当我们发现另一侧（右侧）的条形块高度不是最高的，我们则开始从相反的方向遍历（从右到左）。

此时我们维护左右两个指针，通过两个指针交替进行遍历，实现一次遍历完成。

对于位置left而言，它左边的最大值一定是left_max，右边的最大值大于等于right_max。如果此时left_max < right_max，那么它就能计算出它的位置能存多少水，无论右边将来会不会出现更大的值，都不影响计算出的结果，所以当height[left] < height[right] 时，我们去处理left指针。反之，我们去处理right指针。

**步骤：**
- 初始化left指针为0，right指针为height.length - 1;
- while left < right时：
    - if height[left] < right[right]:
        - left_max = Math.max(left_max, height[left])
        - 累加 left_max - height[left] 到sum中
        - left++
    - else
        - right_max = Math.max(right_max, height[right])
        - sum += right_max - height[right]
        - right--

**实现代码**
```js
var trap = function(height) {
    let sum = 0;
    let left_max = 0;
    let right_max = 0;
    let left = 0;
    let right = height.length - 1;
    while(left < right) {
        if (height[left] < height[right]) {
            left_max = Math.max(left_max, height[left]);
            sum += left_max - height[left];
            left++;
        } else {
            right_max = Math.max(right_max, height[right]);
            sum += right_max - height[right];
            right--;
        }
    }
    return sum;
};
```

**复杂度分析**
- 时间复杂度：O(n)
- 空间复杂度：O(1)



> 如果你觉得这篇内容对你有帮助的话：
>
> 1、**点赞**支持下吧，让更多的人也能看到这篇内容
>
> 2、关注公众号：**前端极客技术(FrontGeek)**，我们一起学习一起进步。