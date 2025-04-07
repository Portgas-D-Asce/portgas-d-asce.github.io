---
layout: page
tags: 数组
image: /assets/image/37.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 拓扑排序（Topo Sort）
categories: [计算机, 数据结构与算法]
---

# 问题描述

最大和子数组（Maximum Sum Subarray）：给定一个一维数组，数组中元素可正可负可为零，求解所有连续子数组的最大和。
# 卡丹尔算法

## 原理

在理解卡丹尔算法前，先明确以下内容：
- 一个长度为n的一维数组，其所有连续子数组的个数为 $n(n + 1) / 2$；
- 将 $n(n + 1) / 2$ 个子数组可分组为：以第1个元素结尾的子数组（1个），以第2个元素结尾的子数组（2个），...，以第n个元素结尾的子数组（n个）
- 最大和子数组为题此时可被理解为：分别计算出以第1个元素结尾的子数组的最大和 $s_1$，以第二个元素结尾的子数组的最大和$s_2$，...，以第 $n$ 个元素结尾的子数组的最大和 $s_n$，则 $s_1$，$s_2$，...，$s_n$ 中的最大值即为最终所求结果；
- 不难理解相邻的 $s_i$ 和 $s_{i+1}$ 之间具有关系：$si+1 = max(s_i + num[i + 1], nums[i + 1])$

卡丹尔算法基于上述思路，遍历数组中元素，先计算出以当前元素结尾的子数组的最大和，然后与临时结果比较，若其值大于临时结果，则用其更新临时结果，否则临时结果不变；完成遍历后，临时结果即为最终结果。

## 代码

```cpp
int maxSubArray(const vector<int>& nums)
{
    int res = numeric_limits<int>::min();
    int temp = -1;
    for(int i = 0; i < nums.size(); ++i)
    {
        temp = temp <= 0 ? nums[i] : temp + nums[i];
        if(temp > res) res = temp;
    }
    return res;
}
```
## 时间/空间复杂度

时间复杂度：O(n)

空间复杂度：O(1)。

# 扩展

若对子数组的长度加一个“长度不能超过k”的约束，即Maximum Sum SubArray 转化为 Maximum sum subarray whose length isn't more than k，该如和求解？

# 练习

[LeetCode 918. Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/)

