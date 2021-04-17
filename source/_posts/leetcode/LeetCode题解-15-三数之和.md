---
title: LeetCode题解|15.三数之和
urlname: leetcode-solution-3sum
tags:
 - LeetCode题解
 - 面试
categories:
 - [算法, LeetCode]
abbrlink: 48923
date: 2020-03-31 22:14:03
---

前面我们解决了[LeetCode题解|1.两数之和]()，下面我们升级版题目：三数之和。

# 题目描述
给定一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。

示例：
```
给定数组 nums = [-1, 0, 1, 2, -1, -4]，

满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```

# 解题思路
在这题中，主要的难点是如何进行去重。

## 暴力法
通过三次遍历去找出所有的解，这种方法的时间复杂度为O(n^3)，并不推荐。

## 排序+双指针

### 算法流程：
1、针对几种特殊情况判断：数组长度n，如果数组为null或者数组长度小于3，返回[]

2、对数组进行升序排序

3、遍历排序后数组：
    - 若nums[i] > 0：因为已经排序好，所以后面不可能会有三个数相加等于0，直接返回结果
    - 对于重复元素：跳过，避免出现重复解
    - 令左指针L = i + 1，右指针 R = n - 1，当L < R时，执行循环。
        - 当 nums[i]+nums[L]+nums[R]=0，执行循环，判断左界和右界是否和下一位置重复，去除重复解。并同时将 L,R 移到下一位置，寻找新的解
        - 若和大于 0，说明 nums[R] 太大，R 左移
        - 若和小于 0，说明 nums[L] 太小，L 右移

### 复杂度分析
时间复杂度：O(n^2)，数组排序 O(NlogN)，遍历数组 O(n)，双指针遍历 O(n)，总体 O(NlogN)+O(n)∗O(n)，O(n^2)
空间复杂度：O(1)

Javascript版本
```javascript
var threeSum = function(nums) {
    let result = [];
    const len = nums.length;
    if (nums == null || len < 3) {
        return result;
    }
    nums.sort((a, b) => a - b);
    for (let i = 0; i < len; i++) {
        if (nums[i] > 0) break;
        if (i > 0 && nums[i] == nums[i - 1]) continue;

        let L = i + 1;
        let R = len - 1;
        while (L < R) {
            const sum = nums[i] + nums[L] + nums[R];
            if (sum == 0) {
                result.push([nums[i], nums[L], nums[R]]);
                while (L < R && nums[L] == nums[L + 1]) {
                    L++;
                }
                while (L <R && nums[R] == nums[R - 1]) {
                    R--;
                }
                L++;
                R--;
            } else if (sum < 0) {
                L++;
            } else {
                R--;
            }
        }
    }
    return result;
};
```

Python版本
```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        
        n=len(nums)
        res=[]
        if(not nums or n<3):
            return []
        nums.sort()
        res=[]
        for i in range(n):
            if(nums[i]>0):
                return res
            if(i>0 and nums[i]==nums[i-1]):
                continue
            L=i+1
            R=n-1
            while(L<R):
                if(nums[i]+nums[L]+nums[R]==0):
                    res.append([nums[i],nums[L],nums[R]])
                    while(L<R and nums[L]==nums[L+1]):
                        L=L+1
                    while(L<R and nums[R]==nums[R-1]):
                        R=R-1
                    L=L+1
                    R=R-1
                elif(nums[i]+nums[L]+nums[R]>0):
                    R=R-1
                else:
                    L=L+1
        return res
```
