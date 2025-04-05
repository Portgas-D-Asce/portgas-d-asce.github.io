---
layout: page
tags: 数据结构与算法
image: /assets/image/6.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: ST 表（Sparse Table）
---

# 概述

ST 表是用于解决 **可重复贡献** 类型区间类问题的数据结构。

- 不支持动态修改
- 支持任意区间查询，且时间复杂度为常数！！！
- 空间复杂度也还不错



使用 ST 表的前提是：满足可重复贡献！！！常见的可重复贡献问题

- 区间最值
- 区间按位和
- 区间按位或
- 区间 GCD



# 构造

ST 表通过动态规划方式构建。



## 状态定义

$$
dp[i][j]：区间 [i, i + 2^j - 1] 的最大值
$$



## 状态转移

对区间进行拆分：

$$
[i, i + 2^j - 1] = [i, i + 2^{j - 1} - 1] + [i + 2^{j - 1}, i + 2^{j - 1} + 2^{j - 1} - 1]
$$

其中：

- $[i, i + 2^j - 1]$ 区间的最大值为 $dp[i][j]$
- $[i, i + 2^{j - 1} - 1]$ 区间的最大值为 $dp[i][j - 1]$
- $[i + 2^{j - 1}, i + 2^{j - 1} + 2^{j - 1} - 1]$ 区间的最大值为 $dp[i + 2^{j - 1}][j - 1]$

故，状态转移方程为：

$$
dp[i][j] = max(dp[i][j - 1], dp[i + 2^{j - 1}][j - 1])
$$



# 查询

设查询区间 $[p, r]$，区间长度 $r - p + 1$。

求 $s$ 使得：

$$
((r - p + 1) / 2) < 2 ^ s <= r - p + 1
$$

使用 $s$ 将查询区间划分为两个长度均为 $2^s$ 的区间（一定具有重叠区域）：

$$
[p, r] = [p, p + 2^s - 1] \cup [r - 2^s + 1, r]
$$

故，查询结果定义为：

$$
max(dp[p][s], dp[r - 2^s + 1][s])
$$



# 实现

```cpp
template<typename T>
class SparseTable {
    using Func = function<T(const T&, const T&)>;
    using vt = vector<T>;
    using vvt = vector<vt>;
private:
    T init_value;
    vvt dp;
    Func op;
public:
    explicit SparseTable(const vt& nums, Func func = Max<T>(), T val = T()) {
        init_value = val;
        op = func;

        size_t n = nums.size();
        ssize_t m = lg(n) + 1;
        dp = vvt(n, vt(m, init_value));

        size_t i = n;
        while(i--) {
            dp[i][0] = nums[i];
            for(size_t j = 1; j < m; ++j) {
                dp[i][j] = dp[i][j - 1];
                size_t k = i + (1 << (j - 1));
                if(k < n) {
                    dp[i][j] = op(dp[i][j], dp[k][j - 1]);
                }
            }
        }
    }
    
    //1-indexed [p, r], same with fenwick tree
    T query(size_t p, size_t r) {
        ssize_t s = lg(r - p + 1);

        return op(dp[p - 1][s], dp[r - (1 << s)][s]);
    }
private:
    ssize_t lg(size_t x) {
        ssize_t z = -1;
        while(x) {
            x >>= 1;
            ++z;
        }
        return z;
    }
};
```



示例

```cpp
// 默认查询区间最大值
SparseTable<int> st(nums);

// 区间最小值
SparseTable<int> st(nums, Min<int>(), INT_MAX);

// 区间 ｜ 值
SparseTable<int> st(nums, bit_or<int>());

// 区间 & 值
SparseTable<int> st(nums, bit_and<int>(), INT_MAX);
```



# 复杂度

时间复杂度：

- 构建复杂度 $O(nlog(n))$
- 查询复杂度 $O(1)$

空间复杂度：$O(nlog(n))$

# 练习

[LeetCode 2736. 最大和查询](https://leetcode.cn/problems/maximum-sum-queries/)

[CodeForces C. Not Equal on a Segment](https://codeforces.com/problemset/problem/622/C)

# 参考

[ST 表 - OI Wiki](https://oi-wiki.org//ds/sparse-table/)
