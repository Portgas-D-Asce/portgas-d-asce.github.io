---
layout: page
tags: 数据结构与算法
image: /assets/image/3.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 二分查找（Binary Search）
---

二分法的本质： **用来求解单调函数的零点！！！**
- 说得更清楚点就是：求解方程 $f(x) = y$ 的根，$f(x)$ 单调， $y$ 表示一个数字。

# 二分法求解方程任意解



## $f(x)$ 以数组形式给出

示例：
- 给定一个单调递增数组 $nums$，判断目标 $tar$ 是否存在；
- 如果存在返回 $tar$ 的下标；否则返回 -1 ；
```cpp
int binary_search(const vector<int> &nums, int n, int tar) {
    int p = 0, r = n - 1;
    while(p <= r) {
        int q = p + r >> 1;
        if(nums[q] == tar) return q;
        nums[q] < tar ? p = q + 1 : r = q - 1;
    }
    return -1;
}
```
理解：
- 数组可以看作是 “自变量是数组下标，因变量为数组中元素” 的一个离散函数，即 $f(x) = nums[x]$；
- 上述问题的本质就是求解方程 $f(x) = nums[x] = tar$ 的解；



## $f(x)$ 为整形函数

示例：
- 判断正整数 $num$ 是否为平方数；
- 如果是平方数，返回 $num$ 的正平方根；否则返回 -1；

```cpp
int f(int x) {
    return x * x;
}

int binary_search(int (*f)(int), int num) {
    int p = 0, r = num >> 1;
    while(p <= r) {
        int q = p + r >> 1;
        int val = f(q);
        if(val == num) return q;
        val < target ? p = q + 1 : r = q - 1;
    }
    return -1;
}
```
本质：求解方程 $f(x) = x * x = num$ 的整数解。



## $f(x)$ 为浮点函数

示例：求解正浮点数 $num$ 的平方根。
```cpp
double f(double x) {
    return x * x;
}

double binary_search(double (*f)(double), double num) {
    double p = 0, r = max(1, num / 2);
    while(r - p > 0.0000001) {
        double q = (p + r) / 2;
        f(q) < num ? p = q : r = q;
    }
    return p;
}
```
本质：求解方程 $f(x) = x * x = num$ 的浮点解。



# 二分法求解最优解

二分最优化问题有两种类型：
- 00001111型：查找第一个 1
- 11110000型：查找最后一个 1

$q$ 如何取值，看下 $p$ 如何赋值就好了

- $p = q + 1$：$q = p + r >> 1$
- $p = q$：$q = p + r + 1 >> 1$



## 00001111型

```cpp
int binary_search01(int *nums, int n) {
    //使用 虚拟上界（r = n） 兼容 00000000 型 情况
    int p = 0, r = n;
    while(p < r) {
        int q = p + r >> 1;
        nums[q] == 1 ? r = q : p = q + 1;
    }
    return p;
}
```



## 11110000型

```cpp
int binary_search10(int *nums, int n) {
    //使用虚拟下界（p = -1） 兼容 00000000型 情况
    int p = -1, r = n - 1;
    while(p < r) {
        int q = p + r + 1 >> 1;
        nums[q] == 1 ? p = q : r = q - 1;
    }
    return p;
}
```



# 应用

## 最大最小



## kth problem

给定一个数 $x$ 排在其前面的数的个数可计算时，则可以通过二分查找来找到排在 $kth$ 位置的数。



有一种场景，”寻找 *** 的 k-th num “ ，对于这种问题，我们可以尝试 x，检查其前面满足条件的数：

- 如果满足条件的数的个数小于 k，则条件为假
- 如果满足条件的数的个数大于等于 k，则条件为真

显然，问题转化为了 00001111 场景的二分查找，寻找第一个 1。



[878. 第 N 个神奇数字](https://leetcode.cn/problems/nth-magical-number/)

[378. 有序矩阵中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-sorted-matrix/)

[668. 乘法表中第k小的数](https://leetcode.cn/problems/kth-smallest-number-in-multiplication-table/)

[3134. 找出唯一性数组的中位数](https://leetcode.cn/problems/find-the-median-of-the-uniqueness-array/)
