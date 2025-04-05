---
layout: page
tags: 数据结构与算法
image: /assets/image/5.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 树状数组（Fenwick Tree）
---

# 原数组 & 前缀和数组 & 差分数组

设有原数组 $nums$:
- 数组 $sum[i] nums[0] + nums[1] + ... + nums[i]$ 称为 $nums$ 的前缀和数组
- 数组 $dif[i] = nums[i] - nums[i - 1]$ 称为 $nums$ 的差分数组

前缀和数组、差分数组 只不过是原数组的不同表示形式。
- 原数组可以在常数时间内访问任意元素！！！
- 前缀和数组可以在常数时间获取任意区间的元素和！！！
- 差分数组可以在常数时间内对任意区间内元素增加/减少固定数值！！！

前缀和、差分互为逆运算：
- 原数组 是 差分数组 的前缀和数组！！！
- 原数组 是 前缀和数组 的差分数组！！！

# 树状数组

![/assets/content/1.png](/assets/content/1.png)

前缀和数组：
- 直接存储了前缀和信息；
- **构建时间复杂度：** $O(n)$；
- **查询原数组任意区间内元素的和：** 时间复杂度 $O(1)$；
- **修改原数组中某个元素的值需要对前缀和数组进行维护：** 时间复杂度为 $O(n)$；

树状数组：
- 间接存储了前缀和信息，被分为连续的多段
- **构建时间复杂度：** $O(n)$；
- **查询原数组任意区间内元素的和：** 实际上是查询两次前缀和，然后作差；时间复杂度 $O(logn)$；
- **修改原数组中某个元素的值需要对树状数组进行维护：** 时间复杂度为 $O(logn)$；



> 树状数组是一种 “兼顾单点修改与区间查询平衡的一种数据结构”



## lowbit 函数

$lowbit$ 是什么？
- $lowbit(x)$：$x$ 的二进制表示最后一位 $1$ 的位权
- $lowbit(x) = x\ \&\ (-x)$

```cpp
x 的二进制表示为：0...1000000
x 的反码为：     1...0111111
x 的补码为：     1...1000000（也就是 -x 的二进制表示）
x & (-x)：      0...1000000
```
## 实现

先上代码，然后再根据代码反向理解：

- 树状数组中每个元素 $ft[i]$ 的含义？
- $lowbit(x)$ 在这期间的作用？

```cpp
//nums 表示原数组，ft 表示 nums 的树状数组
//同前缀和数组，树状数组也需要补充一个前置 0
class FenwickTree{
private:
    vector<int> ft;
    int lowbit(int x) {
        return x & (-x);
    }
public:
    FenwickTree(int n) {
        ft.resize(n + 1, 0);
    }
    //第 idx 个元素 + x
    //nums[idx] + x when 1-indexed
    //nums[idx - 1] + x when 0-indexed
    void add(int idx, int x) {
        while(idx < ft.size()) {
            ft[idx] += x;
            idx += lowbit(idx);
        }
    }
    //前缀和查询
    int sum(int idx) {
        int res = 0;
        while(idx) {
            res += ft[idx];
            idx -= lowbit(idx);
        }
        return res;
    }
};
```

## 理解

树状数组中每个元素 $ft[i]$ 的含义？模拟求和（为了方便描述，这里假设原数组 $nums$ 下标也从 $1$ 开始）：

- $sum[7] = ft[7] + ft[6] + ft[4]$
    - 7 的二进制表示 $111$
    - 实际上，$ft[7] = nums[7]$
    - 实际上，$ft[6] = nums[5] + nums[6]$
    - 实际上，$ft[4] = nums[1] + nums[2] + nums[3] + nums[4]$
- 看了上面例子，对于 $ft[i]$ 的本质基本有感觉了吧 &#x1F914;&#x1F914;
    - $ft[i]$ 表示的是 $nums[i - lowbit(i) + 1] + ... + nums[i]$ 的和！！！
    - $ft[i]$ 维护的是区间 $[i - lowbit(i) + 1, i]$ 的性质

$lowbit(x)$ 在这期间的作用？

- 求和期间：从上到下，快速跳转到下一个求和区间
- 更新期间：从下到上，快速跳转到下一受影响的区间

---

---





# 区间最值

What？？？树状数组可以求任意区间和是可以理解的：求两次前缀和，做差即可得到区间和。树状数组求任意区间最值？！！

一开始也是不相信的，然而事实是，在特定场景下是可行的，而且效果还不错。

特定场景：求区间最小/大值时，要求修改方向只能朝着变小/大的方向修改（线段树是没有这要求的），考虑区间最大值：

- 单点更新下 $nums[5] = 100$，$ft[5]$，$ft[6]$，$ft[8]$ 均受到影响，更新为 $100$，这没问题
- 接着更新下 $nums[5] = 99$，按道理 $ft[5]$，$ft[6]$，$ft[8]$ 均应更新为 $99$，但实际是不会更新的，这正是问题所在！

## 实现

这里以求区间最大值为例，核心思想：

- 更新的时候不仅要维护树状数组 $ft$，还要维护原数组 $nums$
- 查询的时候 $[p, r]$ 最值朴素做法：从起点到终点逐个遍历，取最大值即可，可以使用 $ft$ 对其进行优化
    - 若 $ft[i]$ 所管理的区间在查询区间范围内：用 $ft[i]$ 代表其所管理的区间即可
    - 如果 $ft[i]$ 所管理的区间并没有完全在查询区间内，回到朴素做法即可，用原是数组中的元素做比较即可

示例：查询区间 $[3, 7]$

- $ft[7]$ 管理的范围是 $[7, 7]$，完全在查询范围内
- $ft[6]$ 管理的范围是 $[5, 6]$，完全在查询范围内
- $ft[4]$ 管理的范围是 $[1, 4]$，不完全在查询范围内，使用原数组元素代表 $nums[4]$
- $ft[3]$ 管理的范围是 $[3, 3]$，完全在查询范围内
- $max([3, 7]) = max(ft[7], ft[6], nums[4], ft[3])$

```cpp
//nums is 1-indexed in logical but 0-indexed in fact
class FenwickTree{
private:
    vector<int> ft;
    vector<int> nums;
public:
    FenwickTree(int n) {
        nums = vector<int>(n + 1, INT_MIN);
        ft = vector<int>(n + 1, INT_MIN);
    }

    // require x >= nums[idx]
    void update(int idx, int x) {
        // update origin array
        nums[idx] = x;
        
        // update ft array
        while(idx < ft.size()) {
            ft[idx] = max(ft[idx], x);
            idx += lowbit(idx);
        }
    }

    int query(int p, int r) {
        int res = INT_MIN;
        
        while(p <= r) {
            if(p < r - lowbit(r)) {
                res = max(res, ft[r]);
                r -= lowbit(r);
            } else {
                res = max(res, nums[r]);
                r--;
            }
        }
        return res;
    }

private:
    int lowbit(int x) {
        return x & (-x);
    }
};
```



# 离散化

## 维护离散区间

常规：下标区间是连续的，维护区间 [5, 6] 之间的最值/和

如果下标区间不连续呢：

- 范围小的话，声明一个数组，当成连续的就可以了
- 范围特别大，就没办法声明数组了

解决办法：

- 对出现过的下标进行排序，排完序后，去重
- 每次先用二分查找，查找到对应的连续下标

```cpp
//排序
sort(diff.begin(), diff.end());
//去重
diff.resize(unique(diff.begin(), diff.end()) - diff.begin());
Fenwick fw(diff.size());

long long mx = LLONG_MIN;
for(int i = 0; i < n; ++i) {
    //离散下标转化为连续下标
    int j = lower_bound(diff.begin(), diff.end(), nums[i] - i) - diff.begin();
    long long temp = fw.query(j + 1) + nums[i];
    fw.update(j + 1, temp);
    mx = max(mx, temp);
}
```



# 封装

## 使用说明

支持区间和查询、区间最大值查询、区间最小值查询（前缀  & 任意区间）。前面理论已经介绍过，使用需注意以下情况

- 任意区间查询由于实现手段特殊，效率可能会有所降低
- 用于区间最大/小值查询时，单点更新只能朝着固定方向更新：
    - 最大值查询：新值大于等于旧值
    - 最小值查询：新值小于等于旧值



## 实现

```cpp
template<typename T, template<typename> typename FT = plus,
    template<typename, typename = allocator<T>> typename CT = vector>
class FenwickTree {
private:
    constexpr static FT<T> op = FT<T>();
private:
    const T init_value;
    CT<T> ft;
public:
    explicit FenwickTree(int n, T val = T())
        : init_value(val), ft(n + 1, init_value) {}

    void update(int idx, T x) {
        while(idx < ft.size()) {
            ft[idx] = op(ft[idx], x);
            idx += lowbit(idx);
        }
    }

    // 1-indexed [1, idx]
    T query(int idx) const {
        T res = init_value;
        while(idx) {
            res = op(res, ft[idx]);
            idx -= lowbit(idx);
        }
        return res;
    }

    // 1-indexed [p, r]
    template<typename U>
    T query(int p, int r, const U& nums) const {
        T res = init_value;
        while(p <= r) {
            if(p <= r - lowbit(r)) {
                res = op(res, ft[r]);
                r -= lowbit(r);
            } else {
                res = op(res, nums[r - 1]);
                r--;
            }
        }
        return res;
    }

private:
    static int lowbit(int x) {
        return x & (-x);
    }
};
```



## 示例

```cpp
template<typename T>
class Max {
public:
    T operator()(const T& a, const T& b) const {
        return std::max(a, b);
    }
};

template<typename T>
class Min {
public:
    T operator()(const T& a, const T& b) const {
        return std::min(a, b);
    }
};

// 默认求和，以下两者等价
// 区间和查询
FenwickTree<int> ft_sum(n);
FenwickTree<int, plus> ft_sum(n);

// 区间最大值查询
FenwickTree<int, Min> ft_min(n, INT_MAX);

// 区间最小值查询
FenwickTree<int, Max> ft_max(n, INT_MIN);
```



未验证

```cpp
// 区间乘积
FenwickTree<int, multiplies> ft_mul(n, 1);

// 区间位运算
bit_and;
bit_or;
bit_xor;
```



# 构建

```cpp
explicit FenwickTree(const Cont& nums, T val = T()) {
    init_value = val;
    size_t n = nums.size();
    ft = Cont(n + 1, init_value);

    for(size_t i = 1; i <= n; ++i) {
        ft[i] = op(ft[i], nums[i - 1]);
        size_t idx = i + lowbit(i);
        if(idx <= n) {
            ft[idx] = op(ft[idx], ft[i]);
        }
    }
}
```



# 再次封装

min、max 单向增长

需要满足交换律

```cpp
template<typename Tp, typename Sequence = vector<Tp>,
    typename Calculate = plus<typename Sequence::value_type>>
class FenwickTree {
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_reference const_reference;
    typedef typename Sequence::size_type size_type;
    typedef Sequence container_type;
    typedef Calculate value_calculate;
protected:
    const value_type init_val;
    Sequence raw;
    Sequence ft;
    Calculate cal;
public:
    explicit FenwickTree(size_type n, const_reference init = value_type())
        : init_val(init), raw(n, init), ft(n + 1, init) {}

    explicit FenwickTree(const Sequence& s, const_reference init = value_type())
        : init_val(init), raw(s), ft(s.size() + 1, init) {
        _make_ft();
    }

    template<typename InputIterator>
    FenwickTree(InputIterator first, InputIterator last, const_reference init = value_type())
        : init_val(init), raw(first, last), ft(distance(first, last) + 1, init) {
        _make_ft();
    }

    // 1-indexed idx
    void update(size_type idx, const_reference val) {
        raw[idx - 1] = cal(raw[idx - 1], val);
        while(idx < ft.size()) {
            ft[idx] = cal(ft[idx], val);
            idx += lowbit(idx);
        }
    }

    // 1-indexed idx
    value_type query(size_type idx) const {
        value_type res = init_val;
        while(idx) {
            res = cal(res, ft[idx]);
            idx -= lowbit(idx);
        }
        return res;
    }

    // 1-indexed [p, r]
    value_type query(size_type p, size_type r) const {
        value_type res = init_val;
        while(p <= r) {
            if(p <= r - lowbit(r)) {
                res = cal(res, ft[r]);
                r -= lowbit(r);
            } else {
                res = cal(res, raw[r - 1]);
                r--;
            }
        }
        return res;
    }

private:
    static int lowbit(int x) {
        return x & (-x);
    }

    void _make_ft() {
        size_type n = raw.size();
        for(size_type i = 1; i <= n; ++i) {
            ft[i] = cal(ft[i], raw[i - 1]);
            size_type idx = i + lowbit(i);
            if(idx <= n) {
                ft[idx] = cal(ft[idx], ft[i]);
            }
        }
    }
};
```



# 练习

[LeetCode 2407. 最长递增子序列 II](https://leetcode.cn/problems/longest-increasing-subsequence-ii)



[LeetCode 2926. 平衡子序列的最大和](https://leetcode.cn/problems/maximum-balanced-subsequence-sum/)

[LeetCode 327. 区间和的个数](https://leetcode.cn/problems/count-of-range-sum/)





[LeetCode 1649. Create Sorted Array through Instructions](https://leetcode.com/problems/create-sorted-array-through-instructions/)





# 第三次封装

更新运算？维护运算



基本支持所有 “单点修改，区间查询“

除了最大值，最小值，单向增长。



0-indexed

```cpp
template<typename T>
class Max {
public:
    T operator()(const T& a, const T& b) const {
        return std::max(a, b);
    }
};

template<typename T>
class Min {
public:
    T operator()(const T& a, const T& b) const {
        return std::min(a, b);
    }
};

template<typename Tp, typename Sequence = vector<Tp>,
    typename Calculate = plus<typename Sequence::value_type>>
class FenwickTree {
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_reference const_reference;
    typedef typename Sequence::size_type size_type;
    typedef Sequence container_type;
    typedef Calculate value_calculate;
protected:
    const value_type _init_val;
    container_type _raw, _ft;
    value_calculate _cal;
public:
    explicit FenwickTree(size_type n, const_reference init = value_type())
        : _init_val(init), _raw(n, init), _ft(n + 1, init) {}

    explicit FenwickTree(const container_type& s, const_reference init = value_type())
        : _init_val(init), _raw(s), _ft(s.size() + 1, init) {
        size_type n = _raw.size();
        for(size_type i = 1; i <= n; ++i) {
            _ft[i] = _cal(_ft[i], _raw[i - 1]);
            size_type idx = i + lowbit(i);
            if(idx <= n) {
                _ft[idx] = _cal(_ft[idx], _ft[i]);
            }
        }
    }

    void update(size_type idx, const_reference val) {
        _raw[idx] = _cal(_raw[idx], val);
        idx++;
        while(idx < _ft.size()) {
            _ft[idx] = _cal(_ft[idx], val);
            idx += lowbit(idx);
        }
    }

    value_type query(size_type idx) const {
        idx++;
        value_type res = _init_val;
        while(idx) {
            res = _cal(res, _ft[idx]);
            idx -= lowbit(idx);
        }
        return res;
    }

    value_type query(size_type p, size_type r) const {
        p++, r++;
        value_type res = _init_val;
        while(p <= r) {
            if(p <= r - lowbit(r)) {
                res = _cal(res, _ft[r]);
                r -= lowbit(r);
            } else {
                res = _cal(res, _raw[r - 1]);
                r--;
            }
        }
        return res;
    }

private:
    static inline size_t lowbit(size_t x) {
        return x & (-x);
    }
};
```

缺点，更新仅仅是但方向的，线段树单点更新全能搞得定

- min/max 仅支持单调更新。
- **更新运算** 与 **维护性质运算** 相同。

优点：轻量级，速度快。

## plus

```cpp
FenwickTree<int> ft(n);
FenwickTree<int> ft(n, 0);
```



## multiplies

```cpp
FenwickTree<int> ft(n, 1);
```



## min

仅支持递减更新

```cpp
FenwickTree<int> ft(n, INT_MAX);
```



## max

仅支持递增更新

```cpp
FenwickTree<int> ft(n, INT_MIN);
FenwickTree<int> ft(n, 0);
```



## bit_and

```cpp
FenwickTree<int> ft(n, -1);
```



## bit_or

```cpp
FenwickTree<int> ft(n);
FenwickTree<int> ft(n, 0);
```



## bit_xor

```cpp
FenwickTree<int> ft(n);
FenwickTree<int> ft(n, 0);
```



