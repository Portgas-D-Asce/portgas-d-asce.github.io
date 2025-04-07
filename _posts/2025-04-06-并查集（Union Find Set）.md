---
layout: page
tags: 集论
image: /assets/image/35.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 并查集（Union Find Set）
categories: [计算机, 数据结构与算法]
---

# 简介

并查集：用于处理不相交集合问题的一种数据结构。定义在其上的两个操作：
- **Find：** 查找节点所属于的集合
- **Union：** 合并两个不相交的集合

典型应用当属 Kruskal 算法，优化方式：
- **路径压缩：** 让查找路径上的所有节点均指向根节点，提高下一次查找效率；
- **启发式合并：** 基本没啥用，把 “秩” 小的集合合并到 “秩” 大的集合，使得合并后集合的 “秩” 处于较小状态，有利于查找。

# 实现

```cpp
class UnionFind {
private:
    vector<int> uf;
public:
    UnionFind(int n) {
        uf = vector<int>(n);
        for(int i = 0; i < n; ++i) {
            uf[i] = i;
        }
    }

    int find(int x) {
        return uf[x] == x ? x : uf[x] = find(uf[x]);
    }

    void union1(int x, int y) {
        uf[find(x)] = find(y);
    }
};
```

**注意：** 刚合并完成时，并不是每个元素都知道自己所在的集合的代表是谁！在使用之前，先要确保所有元素都指向自己所在的集合的代表。

```c++
//确保所有元素都知道自己所在的集合的代表是谁
for(int i = 0; i < n; ++i) {
    uf[i] = find(i);
}
//统计 uf 中不同元素个数即可知道最终不相交集合个数
```


# 封装

强行封装成泛型，真是瞎玩！！！！！！！！！！！

```cpp
template<typename T, typename Cont = vector<T>>
class UnionFind {
private:
    Cont uf;
public:
    template<typename U>
    explicit UnionFind(const U& keys) {
        for(auto &key : keys) {
            uf[key] = key;
        }
    }

    explicit UnionFind(T n) {
        uf = Cont(n);
        for(T i = T(); i < n; ++i) {
            uf[i] = i;
        }
    }

    T find(const T &x) {
        return uf[x] == x ? x : uf[x] = find(uf[x]);
    }

    void union1(const T& x, const T& y) {
        uf[find(x)] = find(y);
    }
};
```



示例

```cpp
UnionFind<int> ufx(10);
ufx.union1(0, 7);
cout << ufx.find(0) << endl;

vector<string> ss = {"abc", "xyz"};
UnionFind<string, map<string, string>> uf(ss);
uf.union1("abc", "xyz");
for(const auto& s : ss) {
    cout << s << " : " << uf.find(s) << endl;
}
```
