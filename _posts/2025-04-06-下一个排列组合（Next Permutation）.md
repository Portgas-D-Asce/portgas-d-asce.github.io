---
layout: page
tags: 数组
image: /assets/image/40.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 下一个排列组合（Next Permutation）
categories: [计算机, 数据结构与算法]
---

# 问题描述
对当前数组进行任意排列组合，得到下一个比当前数组大的数组

# 原理
假设数组为 $nums$ 的长度为 $n$, 思路如下：
- 从后往前遍历，找到第一个这样的位置 $i$ 使得 $nums[i] < nums[i + 1]$ （此时，[i + 1, n - 1] 区间上的数是单调递减的）；
- 再从后往前遍历，找到第一个比 $nums[i]$ 大的数 $nums[p]$ ，并交换 $nums[i]$ 和 $nums[p]$ （此时，[i + 1, n - 1]区间上的数还是单调递减的）;
- 最后把 [p, n - 1] 区间上的数变为单递增的。

# 实现
```cpp
void nextPermutation(vector<int>& nums) {
    int n = nums.size(), p = n;
    for(int i = n - 1; i > 0; --i) {
        if(nums[i - 1] >= nums[i]) continue;
        p = i - 1;
        break;
    }
    for(int i = n - 1; i > p; --i) {
        if(nums[p] >= nums[i]) continue;
        swap(nums[p], nums[i]);
        break;
    }
    
    if(p == n) p = -1;
    for(int i = p + 1, j = n - 1; i < j; ++i, --j) {
        swap(nums[i], nums[j]);
    }
}
```

