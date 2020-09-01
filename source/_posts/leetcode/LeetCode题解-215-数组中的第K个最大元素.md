---
title: LeetCode题解|215.数组中的第K个最大元素
urlname: kth-largest-element-in-an-array
tags:
  - LeetCode题解
  - 面试
  - 排序
categories:
  - LeetCode
  - 算法
abbrlink: 3399
date: 2020-09-01 23:12:14
---


## 前言
前面我们一起学过十种常见的排序算法，我们一起来看一道和排序有关的LeetCode题目：215.数组中的第K个最大元素，题目链接：https://leetcode-cn.com/problems/kth-largest-element-in-an-array/。


## 题目描述
在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

**示例 1:**
```
输入: [3,2,1,5,6,4] 和 k = 2
输出: 5
```

## 解法1：Array.sort()
求解Kth Element问题，我们可以利用Array.sort()方法对数组进行升序排序，然后返回倒数第k个元素即可。

实现代码如下：
```javascript
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var findKthLargest = function(nums, k) {
    nums = nums.sort((a, b) => a - b);
    return nums[nums.length - k];
};
```

> 注意：sort方法如果没有指定排序函数的话，元素按照转换为的字符串的各个字符的Unicode位点进行排序。

## 解法2：堆排序
堆排序可以用于求解TopK Element问题，也就是K个最小元素的问题。可以维护一个大小为K的最小堆，最小堆中的元素就是最小元素。最小堆需要使用大顶堆来实现，大顶堆表示堆顶元素是堆中最大元素。这是因为我们要得到 k 个最小的元素，因此当遍历到一个新的元素时，需要知道这个新元素是否比堆中最大的元素更小，更小的话就把堆中最大元素去除，并将新元素添加到堆中。所以我们需要很容易得到最大元素并移除最大元素，大顶堆就能很好满足这个要求。

堆也可以用于求解 Kth Element 问题，得到了大小为 k 的最小堆之后，因为使用了大顶堆来实现，因此堆顶元素就是第 k 大的元素。


我们来看下这道题用堆排序的实现：
```javascript
let findKthLargest = function(nums, k) {
    // 从nums中取出前k个数，构建一个小顶堆
    let heap = [,], i = 0
    while (i < k) {
        heap.push(nums[i++])
    }
    buildHeap(heap, k)

    // 从k位开始遍历数组
    for (let i = k; i < nums.length; i++) {
        if (heap[1] < nums[i]) {
            // 替换并堆化
            heap[1] = nums[i]
            heapify(heap, k, 1)
        }
    }
    return heap[1]
}

// 原地建堆，从后往前，自上而下式建小顶堆
let buildHeap = function (arr, k) {
    if (k === 1) return
    // 从最后一个非叶子节点开始，自上而下式堆化
    for (let i = Math.floor(k / 2); i >= 1; i--) {
        heapify(arr, k, i)
    }
}

// 堆化
let heapify = function (arr, k, i) {
    while (true) {
        let minIndex = i
        if (2 * i <= k && arr[2 * i] < arr[i]) {
            minIndex = 2 * i
        }
        if (2 * i + 1 <= k && arr[2 * i +1] < arr[minIndex]) {
            minIndex = 2 * i + 1
        }
        if (minIndex !== i) {
            swap(arr, i, minIndex)
            i = minIndex
        } else {
            break
        }
    }
}

// 交换
let swap = (arr, i , j) => {
    let temp = arr[i]
    arr[i] = arr[j]
    arr[j] = temp
}
```

复杂度分析：
- 时间复杂度：O(NlogK)
- 空间复杂度：O(K)

## 解法3：快速排序
快速排序可以用于求解Kth Element问题，也就是第K个元素的问题。

快速排序也可以求解TopK Element问题，因为找到Kth Element之后，再遍历一次数组，所有小于等于Kth Element的元素都是TopK Elements。


我们来看下使用快速排序解决这道题的实现代码：
```javascript
let findKthLargest = function(nums, k) {
    return quickSelect(nums, nums.length - k)
};

let quickSelect = (arr, k) => {
  return quick(arr, 0 , arr.length - 1, k)
}

let quick = (arr, left, right, k) => {
  let index
  if(left < right) {
    // 划分数组
    index = partition(arr, left, right)
    // Top k
    if(k === index) {
        return arr[index]
    } else if(k < index) {
        // Top k 在左边
        return quick(arr, left, index-1, k)
    } else {
        // Top k 在右边
        return quick(arr, index+1, right, k)
    }
  }
  return arr[left]
}

let partition = (arr, left, right) => {
  // 取中间项为基准
  var datum = arr[Math.floor(Math.random() * (right - left + 1)) + left],
      i = left,
      j = right
  // 开始调整
  while(i < j) {
    
    // 左指针右移
    while(arr[i] < datum) {
      i++
    }
    
    // 右指针左移
    while(arr[j] > datum) {
      j--
    }
    
    // 交换
    if(i < j) swap(arr, i, j)

    // 当数组中存在重复数据时，即都为datum，但位置不同
    // 继续递增i，防止死循环
    if(arr[i] === arr[j] && i !== j) {
        i++
    }
  }
  return i
}

// 交换
let swap = (arr, i , j) => {
    let temp = arr[i]
    arr[i] = arr[j]
    arr[j] = temp
}
```

复杂度分析：
- 时间复杂度：O(N)
- 空间复杂度：O(1)