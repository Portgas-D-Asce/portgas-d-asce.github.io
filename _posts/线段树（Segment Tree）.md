# 概述

## 优势

线段树是区间问题的重武器：

- 对于区间最值问题，不要求修改是单调的（是通过回溯方式进行更新的）。
- 支持 $O(log(n))$ 复杂度的区间修改



## 4n vs 2n

4n 线段树是自顶向下生成的线短树，大概长什么样子？

- 满二叉树？完全二叉树？平衡二叉树？说是平衡二叉树，不够精细。**准确说是介于平衡二叉树和完全二叉树之间：倒数第二层是一棵满二叉树，最后一层叶节点可能随机出现**
- 叶子节点中存储着数组原数据，非叶子节点存储着数组区间信息
- 4n 情况出现在：最后一层只有两个叶节点，且位于最右侧，此时需要 4n 的存储空间



2n 线短树是自下而上生成的线短树，暂不讨论。



# 单点修改

支持区间和、区间最大/小值

```cpp
template<typename T, template<typename> typename FT = plus,
        template<typename, typename = allocator<T>> typename CT = vector>
class SegmentTree {
private:
    constexpr static FT<T> op = FT<T>();
private:
    const T init_value;
    CT<T> seg;
public:
    explicit SegmentTree(int n, T val = T())
        : init_value(val), seg(n << 2, val) {}

    // pos is 1-indexed
    void update(int p, T val) {
        int n = seg.size() >> 2;
        if(p <= 0) return;
        // 线段树 1 号节点，维护整个数组信息
        update(1, 0, n - 1, p - 1, val);
    }

    // p, r is 1-indexed
    T query(int p, int r) {
        int n = seg.size() >> 2;
        if(p > r || r <= 0) return init_value;
        return query(1, 0, n - 1, p - 1, r - 1);
    }
private:
    int left(int idx) { return idx << 1; }
    int right(int idx) { return (idx << 1) | 1; }
    void update(int idx) {
        seg[idx] = op(seg[left(idx)], seg[right(idx)]);
    }

    //idx 线段树节点下标, [p, r] 维护区间, x 更新位置 0-indexed, val 增量/修改后的值
    void update(int idx, int p, int r, int x, T val) {
        // 到达叶节点
        if(p == r) {
            seg[idx] = val;
            return;
        }

        int q = (p + r) >> 1;
        if(x <= q) {
            // 更新位置与左半维护区间有交集
            update(left(idx), p, q, x, val);
        } else {
            // 更新位置与右半维护区间有交集
            update(right(idx), q + 1, r, x, val);
        }
        // 回溯
        update(idx);
    }

    //idx 线段树节点下标, [p, r] 维护区间, [x, y]查询区间 0-indexed
    T query(int idx, int p, int r, int x, int y) {
        // 维护区间在查询区间范围内
        if(x <= p && r <= y) return seg[idx];

        int q = (p + r) >> 1;
        T res = init_value;
        if(x <= q) {
            // 查询区间与左半维护区间存在交集
            res = op(res, query(left(idx), p, q, x, y));
        }

        if(y > q) {
            // 查询区间与右半维护区间存在交集
            res = op(res, query(right(idx), q + 1, r, x, y));
        }
        return res;
    }
};
```



# 区间修改

## 原理

为了支持快速区间修改，线短树引入 **懒标记**

- 懒标记：区间修改时，只是进行简单标记，并不将修改实际生效到每个元素；在查询前，会将标记下沉，使得修改生效。
- 不使用懒标记的区间修改时间复杂度为线性，使用后可达到 $O(log(n))$ 



## 实现

支持区间和、区间最大/小值

```cpp
template<template<typename> typename MFT>
struct Magic {
    template<typename U>
    static U total(U val, int x) {
        return val;
    }
};

template<>
struct Magic<plus> {
    template<typename U>
    static U total(U val, int x) {
        return val * x;
    }
};
// 还可以偏特化 multi 等

template<typename T, template<typename> typename FT = plus,
    template<typename, typename = allocator<T>> typename CT = vector>
class SegmentTree {
private:
    constexpr static FT<T> op = FT<T>();
private:
    const T init_value;
    //线段树
    CT<T> seg;
    //懒标记
    CT<T> tag;
public:
    explicit SegmentTree(int n, T val = T())
        : init_value(val), seg(n << 2, val), tag(n << 2, val) {}

    // x, y is 1-indexed
    void update(int p, int r, T val) {
        int n = seg.size() >> 2;
        if(p > r || r <= 0) return;
        update(1, 0, n - 1, p - 1, r - 1, val);
    }

    // x, y is 1-indexed
    T query(int p, int r) {
        int n = seg.size() >> 2;
        if(p > r || r <= 0) return init_value;
        return query(1, 0, n - 1, p - 1, r - 1);
    }
private:
    int left(int idx) { return idx << 1; }
    int right(int idx) { return idx << 1 | 1; }
    int mid(int p, int r) { return (p + r) >> 1; }
    // idx 线段树节点下标
    void update(int idx) { seg[idx] = op(seg[left(idx)], seg[right(idx)]); }

    T magic(int x, T val) {
        return Magic<FT>::total(val, x);
    }

    // 懒标记下沉, idx 线段树节点下标 [p, r] 维护区间 0-indexed
    void down(int idx, int p, int r) {
        if(tag[idx] == init_value) return;

        int q = mid(p, r);
        // 左维护区间下沉
        seg[left(idx)] = op(seg[left(idx)], magic(q - p + 1, tag[idx]));
        tag[left(idx)] = op(tag[left(idx)], tag[idx]);

        // 右维护区间下沉
        seg[right(idx)] = op(seg[right(idx)], magic(r - q, tag[idx]));
        tag[right(idx)] = op(tag[right(idx)], tag[idx]);

        tag[idx] = init_value;
    }

    //idx 线段树节点下标 [p, r] 维护区间, [x, y] 更新区间 0-indexed z 增量
    void update(int idx, int p, int r, int x, int y, T val) {
        // 节点维护区间在更新区间范围内
        if(x <= p && r <= y) {
            seg[idx] = op(seg[idx], magic(r - p + 1, val));
            tag[idx] = op(tag[idx], val);
            return;
        }

        // 懒标记下沉
        down(idx, p, r);

        int q = mid(p, r);
        if(x <= q) {
            // 更新区间和左半维护区间存在交集
            update(left(idx), p, q, x, y, val);
        }

        if(y > q) {
            // 更新区间和右半维护区间存在交集
            update(right(idx), q + 1, r, x, y, val);
        }
        // 回溯
        update(idx);
    }

    T query(int idx, int p, int r, int x, int y) {
        // 节点维护区间在查询区间范围内
        if(x <= p && r <= y) return seg[idx];

        // 懒标记下沉
        down(idx, p, r);

        int q = mid(p, r);
        T res = init_value;
        if(x <= q) {
            // 查询区间和左半维护区间存在交集
            res = op(res, query(left(idx), p, q, x, y));
        }

        if(y > q) {
            // 查询区间和右半维护区间存在交集
            res = op(res, query(right(idx), q + 1, r, x, y));
        }
        return res;
    }
};
```



为什么在 `update` 的时候也需要懒标记下沉？？

- `update` 回溯的时候会更新区间信息。为什么回溯的时候需要 `update` ？？？
- 没有下沉会导致左右子树的信息缺失，最终导致回溯到父节点信息缺失。



示例

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

// 默认区间和
SegmentTree<int> seg(nums.size());
// 区间最小值
SegmentTree<int, Min> seg(nums.size(), INT_MAX);
// 区间最大值
SegmentTree<int, Max> seg(nums.size(), INT_MIN);

// 初始化
for(int i = 0; i < nums.size(); ++i) {
    seg.update(i + 1, i + 1, nums[i]);
}

// 查询
int res = seg.query(1, 5);
```



# 附

```cpp
//nums:原数组，[p, r]:对哪个区间建线段树；seg:线段树；idx:线段树的节点索引；
void build(vector<int> &nums, int idx, int p, int r) {
    if(p == r) {
        seg[idx] = nums[p];
        return;
    }
    int q = (p + r) >> 1;
    build(nums, left(idx), p, q);
    build(nums, right(idx), q + 1, r);
    update(idx);
    //tag[idx] = init_value;
}
```



# 练习

[LeetCode 2407. 最长递增子序列 II](https://leetcode.cn/problems/longest-increasing-subsequence-ii)

[P3372 【模板】线段树 1](https://www.luogu.com.cn/problem/P3372)

[2569. 更新数组后处理求和查询](https://leetcode.cn/problems/handling-sum-queries-after-update/)



# 封装

0 indexed

```cpp
template<typename Tp, typename Sequence = vector<Tp>,
    typename Calculate = plus<typename Sequence::value_type>,
    typename Accumulate = Accumulate<Calculate>>
class SegmentTree {
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_reference const_reference;
    typedef typename Sequence::size_type size_type;
    typedef Sequence container_type;
    typedef Calculate value_calculate;
private:
    const value_type init_val;
    //懒标记
    container_type tag;
    //线段树
    container_type seg;
    Calculate cal;
    Accumulate acc;
public:
    explicit SegmentTree(size_type n, const_reference val = value_type())
        : init_val(val), tag(n << 2, val), seg(n << 2, val) {}

    explicit SegmentTree(const Sequence& s, const_reference val = value_type())
        : init_val(val), tag(s.size() << 2, val), seg(s.size() << 2, val) {
        size_type n = s.size();
        for(size_type i = 1; i <= n; ++i) {
            update(i, i, s[i]);
        }
    }

    template<typename InputIterator>
    explicit SegmentTree(InputIterator first, InputIterator last, const_reference val = value_type())
        : init_val(val), tag(distance(first, last) << 2, val),
        seg(distance(first, last) << 2, val) {
        for(size_type i = 1; first != last; ++i, ++first) {
            update(i, i, *first);
        }
    }

    void update(size_type p, size_type r, const_reference val) {
        int n = seg.size() >> 2;
        update(1, 0, n - 1, p, r, val);
    }

    value_type query(size_type p, size_type r) {
        int n = seg.size() >> 2;
        return query(1, 0, n - 1, p, r);
    }
private:
    inline static size_type left(size_type idx) { return idx << 1; }
    inline static size_type right(size_type idx) { return idx << 1 | 1; }
    inline static size_type mid(size_type p, size_type r) { return (p + r) >> 1; }
    // idx 线段树节点下标
    void update(size_type idx) {
        seg[idx] = cal(seg[left(idx)], seg[right(idx)]);
    }

    // 懒标记下沉, idx 线段树节点下标 [p, r] 维护区间 0-indexed
    void down(size_type idx, size_type p, size_type r) {
        if(tag[idx] == init_val) return;

        int q = mid(p, r);
        // 左维护区间下沉
        seg[left(idx)] = cal(seg[left(idx)], acc(q - p + 1, tag[idx]));
        tag[left(idx)] = cal(tag[left(idx)], tag[idx]);

        // 右维护区间下沉
        seg[right(idx)] = cal(seg[right(idx)], acc(r - q, tag[idx]));
        tag[right(idx)] = cal(tag[right(idx)], tag[idx]);

        tag[idx] = init_val;
    }

    //idx 线段树节点下标 [p, r] 维护区间, [x, y] 更新区间 0-indexed z 增量
    void update(size_type idx, size_type p, size_type r, size_type x, size_type y, const_reference val) {
        // 节点维护区间在更新区间范围内
        if(x <= p && r <= y) {
            seg[idx] = cal(seg[idx], acc(r - p + 1, val));
            tag[idx] = cal(tag[idx], val);
            return;
        }

        // 懒标记下沉
        down(idx, p, r);

        int q = mid(p, r);
        if(x <= q) {
            // 更新区间和左半维护区间存在交集
            update(left(idx), p, q, x, y, val);
        }

        if(y > q) {
            // 更新区间和右半维护区间存在交集
            update(right(idx), q + 1, r, x, y, val);
        }
        // 回溯
        update(idx);
    }

    value_type query(size_type idx, size_type p, size_type r, size_type x, size_type y) {
        // 节点维护区间在查询区间范围内
        if(x <= p && r <= y) return seg[idx];

        // 懒标记下沉
        down(idx, p, r);

        size_type q = mid(p, r);
        value_type res = init_val;
        if(x <= q) {
            // 查询区间和左半维护区间存在交集
            res = cal(res, query(left(idx), p, q, x, y));
        }

        if(y > q) {
            // 查询区间和右半维护区间存在交集
            res = cal(res, query(right(idx), q + 1, r, x, y));
        }
        return res;
    }
};

```



## default

默认累积函数：适用于 min、max

bit_and、bit_or ？？？？

懒标记：

- max(x, y, z) = max(x, max(y, z))
- x & y & z = x & (y & z)
- x | y | z = x | (y | z)

```cpp
template<typename Tp>
class Accumulate {
public:
    Tp operator()(const Tp& val, const Tp& x) const {
        return val;
    }
};
```



## plus

懒标记 x + y + z = x + (y + z)

```cpp
template<typename Tp>
class Accumulate<plus<Tp>> {
public:
    Tp operator()(const Tp& val, const Tp& x) const {
        return val * x;
    }
};
```



## bit_xor

懒标记 x ^ y ^ z = x ^ (y ^ z)

```cpp
template<typename Tp>
class Accumulate<bit_xor<Tp>> {
public:
    Tp operator()(const Tp& val, const Tp& x) const {
        return (x & 1) ? val : 0;
    }
};
```



## multiplies

懒标记 x * y * z = x * (y * z)

```cpp
template<typename Tp>
class Accumulate<multiplies<Tp>> {
public:
    Tp operator()(const Tp& val, const Tp& x) const {
        //return qpow(val, x);
    }
};
```





# 第三次封装

## 单点更新

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
    typename Calculate = Max<typename Sequence::value_type>>
class SegmentTree {
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_reference const_reference;
    typedef typename Sequence::size_type size_type;
    typedef Sequence container_type;
private:
    const value_type init_val;
    container_type seg;
    Calculate cal;
public:
    explicit SegmentTree(size_type n, const_reference val = value_type())
        : init_val(val), seg(n << 2, val) {}

    explicit SegmentTree(const Sequence& s, const_reference val = value_type())
        : init_val(val), seg(s.size() << 2, val) {
        size_type n = s.size();
        for(size_type i = 0; i < n; ++i) {
            update(i, s[i]);
        }
    }

    template<typename InputIterator>
    explicit SegmentTree(InputIterator first, InputIterator last, const_reference val = value_type())
        : init_val(val), seg(distance(first, last) << 2, val) {
        for(size_type i = 0; first != last; ++i, ++first) {
            update(i, *first);
        }
    }

    void update(size_type p, const_reference val) {
        int n = seg.size() >> 2;
        update(1, 0, n - 1, p, val);
    }

    value_type query(size_type p, size_type r) {
        int n = seg.size() >> 2;
        return query(1, 0, n - 1, p, r);
    }
private:
    inline static size_type left(size_type idx) { return idx << 1; }
    inline static size_type right(size_type idx) { return idx << 1 | 1; }
    inline static size_type mid(size_type p, size_type r) { return (p + r) >> 1; }
    // idx 线段树节点下标
    void update(size_type idx) {
        seg[idx] = cal(seg[left(idx)], seg[right(idx)]);
    }

    void update(size_type idx, size_type p, size_type r, size_type x, const_reference val) {
        // 到达叶节点
        if(p == r) {
            seg[idx] = val;
            // seg[idx] = cal(seg[idx], val);
            // seg[idx] = sth_cal(seg[idx], val)
            return;
        }

        size_type q = (p + r) >> 1;
        if(x <= q) {
            // 更新位置与左半维护区间有交集
            update(left(idx), p, q, x, val);
        } else {
            // 更新位置与右半维护区间有交集
            update(right(idx), q + 1, r, x, val);
        }
        // 回溯
        update(idx);
    }

    value_type query(size_type idx, size_type p, size_type r, size_type x, size_type y) {
        // 维护区间在查询区间范围内
        if(x <= p && r <= y) return seg[idx];

        size_type q = (p + r) >> 1;
        value_type res = init_val;
        if(x <= q) {
            // 查询区间与左半维护区间存在交集
            res = cal(res, query(left(idx), p, q, x, y));
        }

        if(y > q) {
            // 查询区间与右半维护区间存在交集
            res = cal(res, query(right(idx), q + 1, r, x, y));
        }
        return res;
    }
};
```

优势主要是因为采用回溯更新：

- 支持 min/max 任意方向增长
- 支持 **维护性质运算** 与 **更新运算** 不同：例如，单点修改是增加 x，维护的性质是异或和。

### min

```cpp
SegmentTree<int> st(n, INT_MAX);
```



### max

```cpp
SegmentTree<int> st(n, INT_MIN);
SegmentTree<int> st(n, 0);
```



### others

注意 update 处使用的是赋值，而非运算结果。



## 区间更新

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
    typename Calculate = Max<typename Sequence::value_type>>
class SegmentTree {
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_reference const_reference;
    typedef typename Sequence::size_type size_type;
    typedef Sequence container_type;
private:
    const value_type init_val;
    //懒标记
    container_type tag;
    //线段树
    container_type seg;
    Calculate cal;
public:
    explicit SegmentTree(size_type n, const_reference val = value_type())
        : init_val(val), tag(n << 2, val), seg(n << 2, val) {}

    explicit SegmentTree(const Sequence& s, const_reference val = value_type())
        : init_val(val), tag(s.size() << 2, val), seg(s.size() << 2, val) {
        size_type n = s.size();
        for(size_type i = 0; i < n; ++i) {
            update(i, i, s[i]);
        }
    }

    template<typename InputIterator>
    explicit SegmentTree(InputIterator first, InputIterator last, const_reference val = value_type())
        : init_val(val), tag(distance(first, last) << 2, val),
        seg(distance(first, last) << 2, val) {
        for(size_type i = 0; first != last; ++i, ++first) {
            update(i, i, *first);
        }
    }

    void update(size_type p, size_type r, const_reference val) {
        int n = seg.size() >> 2;
        update(1, 0, n - 1, p, r, val);
    }

    value_type query(size_type p, size_type r) {
        int n = seg.size() >> 2;
        return query(1, 0, n - 1, p, r);
    }
private:
    inline static size_type left(size_type idx) { return idx << 1; }
    inline static size_type right(size_type idx) { return idx << 1 | 1; }
    inline static size_type mid(size_type p, size_type r) { return (p + r) >> 1; }
    // idx 线段树节点下标
    void update(size_type idx) {
        seg[idx] = cal(seg[left(idx)], seg[right(idx)]);
    }

    // 懒标记下沉, idx 线段树节点下标 [p, r] 维护区间 0-indexed
    void down(size_type idx, size_type p, size_type r) {
        if(tag[idx] == init_val) return;

        int q = mid(p, r);
        // 左维护区间下沉
        seg[left(idx)] = update_seg(seg[left(idx)], tag[idx], p, q);
        tag[left(idx)] = update_tag(tag[left(idx)], tag[idx]);

        // 右维护区间下沉
        seg[right(idx)] = update_seg(seg[right(idx)], tag[idx], q + 1, r);
        tag[right(idx)] = update_tag(tag[left(idx)], tag[idx]);

        tag[idx] = init_val;
    }

    //idx 线段树节点下标 [p, r] 维护区间, [x, y] 更新区间 0-indexed z 增量
    void update(size_type idx, size_type p, size_type r, size_type x, size_type y, const_reference val) {
        // 节点维护区间在更新区间范围内
        if(x <= p && r <= y) {
            seg[idx] = update_seg(seg[idx], val, p, r);
            tag[idx] = update_tag(tag[idx], val);
            return;
        }

        // 懒标记下沉
        down(idx, p, r);

        int q = mid(p, r);
        if(x <= q) {
            // 更新区间和左半维护区间存在交集
            update(left(idx), p, q, x, y, val);
        }

        if(y > q) {
            // 更新区间和右半维护区间存在交集
            update(right(idx), q + 1, r, x, y, val);
        }
        // 回溯
        update(idx);
    }

    value_type query(size_type idx, size_type p, size_type r, size_type x, size_type y) {
        // 节点维护区间在查询区间范围内
        if(x <= p && r <= y) return seg[idx];

        // 懒标记下沉
        down(idx, p, r);

        size_type q = mid(p, r);
        value_type res = init_val;
        if(x <= q) {
            // 查询区间和左半维护区间存在交集
            res = cal(res, query(left(idx), p, q, x, y));
        }

        if(y > q) {
            // 查询区间和右半维护区间存在交集
            res = cal(res, query(right(idx), q + 1, r, x, y));
        }
        return res;
    }
    static inline Tp update_tag(const_reference old, const_reference nw) {
        // 累积类型: sum, bit_xor, bit_and, bit_or
        // return cal(old, nw);
        // 设置类型: max, min
        return nw;
    }
    static inline Tp update_seg(const_reference old, const_reference val, size_type p, size_type r) {
        // sum
        // return cal(old, (r - p + 1) * val);
        // bit_xor
        // return cal(old, ((r - p + 1) & 1) ? val, 0);
        // bit_and, bit_or
        // 设置类型: max, min
        return val;
    }
};
```



### min/max

```cpp
static inline Tp update_tag(const_reference old, const_reference nw) {
    return nw;
}
static inline Tp update_seg(const_reference old, const_reference val, size_type p, size_type r) {
    return val;
}

SegmentTree<int> st(n, INT_MAX);
SegmentTree<int> st(n, INT_MIN);
SegmentTree<int> st(n);
SegmentTree<int> st(n, 0);
```

### plus

```cpp
static inline Tp update_tag(const_reference old, const_reference nw) {
    return cal(old, nw);
}
static inline Tp update_seg(const_reference old, const_reference val, size_type p, size_type r) {
    return cal(old, val * (r - p + 1));
}

SegmentTree<int> st(n);
SegmentTree<int> st(n, 0);
```



### bit_and

```cpp
static inline Tp update_tag(const_reference old, const_reference nw) {
    return cal(old, nw);
}
static inline Tp update_seg(const_reference old, const_reference val, size_type p, size_type r) {
    return cal(old, val);
}

SegmentTree<int> st(n, -1);
```



### bit_or

```cpp
static inline Tp update_tag(const_reference old, const_reference nw) {
    return cal(old, nw);
}
static inline Tp update_seg(const_reference old, const_reference val, size_type p, size_type r) {
    return cal(old, val);
}

SegmentTree<int> st(n);
SegmentTree<int> st(n, 0);
```



### bit_xor

```cpp
static inline Tp update_tag(const_reference old, const_reference nw) {
    return cal(old, nw);
}
static inline Tp update_seg(const_reference old, const_reference val, size_type p, size_type r) {
    val = (r - p & 1) ? 0 : val;
    return cal(old, val);
}

SegmentTree<int> st(n);
SegmentTree<int> st(n, 0);
```



### multiplies

```cpp
static inline Tp update_tag(const_reference old, const_reference nw) {
    return cal(old, nw);
}
static inline Tp update_seg(const_reference old, const_reference val, size_type p, size_type r) {
    return cal(old, pow(val, r - p + 1));
}

SegmentTree<int> st(n, 1);
```







