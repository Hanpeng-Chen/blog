---
title: LeetCode题解|1.两数之和
urlname: leetcode-solution-two-sum
abbrlink: 41236
date: 2020-03-30 09:01:09
tags:
 - LeetCode题解
 - 面试
categories:
 - [算法, LeetCode]
---

# 题目描述
给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例:
```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

# 解题思路
## 暴力法
暴力法很简单，遍历数组nums中的每个元素x，并通过遍历剩余部分查找是否存在一个值与 target-x 相等。下面是暴力法的代码：
### python版本
```python
def twoSum(nums, target):
    lens = len(nums)
    for i, value in enumerate(nums):
        diff = target - value
        j = i + 1
        while j < len:
            if nums[j] == diff
                return [i, j]
            j += 1
    return []
```

### javascript版本
```javascript
var twoSum = function(nums, target) {
    for (var i = 0; i < nums.length; i++) {
        var diff = target - nums[i]
        for (var j = i + 1; j < nums.length; j++) {
            if (diff === nums[j]) {
                return [i, j]
            }
        }
    }
    return []
};
```

我们来看一下暴力法的复杂度：
- 时间复杂度：O(n^2)
- 空间复杂度：O(1)

暴力法实现起来很简单，但时间复杂度很高，容易出现超出时间限制的问题。

## 哈希表
我们可以对运行时间复杂度进行优化，因此我们需要一种更有效的方法来检查数组中是否存在目标元素。如果存在，我们需要找出它的索引。保持数组中的每个元素与其索引相互对应的最好方法是什么？哈希表。

通过以空间换取速度的方式，我们可以将查找时间从 O(n) 降低到 O(1)。哈希表正是为此目的而构建的，它支持以 近似 恒定的时间进行快速查找。我用“近似”来描述，是因为一旦出现冲突，查找用时可能会退化到 O(n)。但只要你仔细地挑选哈希函数，在哈希表中进行查找的用时应当被摊销为 O(1)。

一个简单的实现使用了两次迭代。在第一次迭代中，我们将每个元素的值和它的索引添加到表中。然后，在第二次迭代中，我们将检查每个元素所对应的目标元素（target - nums[i]）是否存在于表中。注意，该目标元素不能是 nums[i] 本身！

但是我们可以一次完成。在进行迭代并将元素插入到表中的同时，我们还会回过头来检查表中是否已经存在当前元素所对应的目标元素。如果它存在，那我们已经找到了对应解，并立即将其返回。

复杂度分析：
- 时间复杂度：O(n)，我们只遍历了包含有 nn 个元素的列表一次。在表中进行的每次查找只花费 O(1) 的时间。
- 空间复杂度：O(n)，所需的额外空间取决于哈希表中存储的元素数量，该表最多需要存储 n 个元素。

下面我们来看下Python和JavaScript两种语言的实现代码：

### Python版本
在Python中我们可以通过字典来模拟哈希查询的过程。

Python代码实现：
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        hasMap = {}
        for i, num in enumerate(nums):
            if (target - num) in hasMap:
                return [hasMap[target - num], i]
            hasMap[num] = i
        return []
```

### JavaScript版本
在JavaScript中，可以通过对象来模拟哈希查询的过程。

```JavaScript
var twoSum = function(nums, target) {
    var dict = {}
    for (var i = 0; i < nums.length; i++) {
        if (dict.hasOwnProperty(target - nums[i])) {
            return [dict[target - nums[i]], i]
        }
        dict[nums[i]] = i
    }
};
```