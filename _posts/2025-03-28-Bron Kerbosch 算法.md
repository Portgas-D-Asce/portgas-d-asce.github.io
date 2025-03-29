---
layout: page
tags: 数据结构与算法 图论
image: /assets/image/17.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: Bron Kerbosch 算法
---

# 原理

如何找到最大团？寻找所有极大团，期间保存最大团！



实现方式：递归 + 减枝，主要是减枝！！！



# 朴素算法

实现时需要维护两个集合：

- **$R$ 集合：** 保存当前团节点

- **$P$ 集合：** 当前团的邻接点集。该集合为空时，说明找到一个极大团



寻找极大团步骤：

- 先将第一个点 $v_1$ 加入 **团 $R$ **，$v_1$ 的邻接点集 $X_1$ 作为团 $R$ 的 **邻接节点集 $P$**
- 从 $P$ 中选择一个点 $v_2$，作为 $R$ 的第二个点
- 用 $v_2$ 的邻接点集 $X_2$，更新 $P = P \cap X_2$
- 从 $P$ 中再选择一个点 $v_3$，作为 $R$ 的第三个点
- 用 $v_3$ 的邻接点集 $X_3$，更新 $P = P \cap X_3$
- ...
- 直到团的邻接点集 $P$ 为空，说明找到了一个极大团
- 回溯，接着找其它极大团



```cpp
void max_clique(set<int> &r, set<int> &p, set<int> &clique) {
    // p 集合为空，说明找到了一个极大团
    if(p.empty()) {
        if(r.size() > clique.size()) {
            clique = r;
        }
        return;
    }
    
    for(int u : p) {
        if(r.find(u) != r.end()) continue;
        // 将 u 加入到团 r
        r.insert(u);
        
        // P 与 u 的邻接点集求交集 np
        set<int> np;
        for(int v : g[u]) {
            if(p.find(v) != p.end()) {
                np.insert(v);
            }
        }
        max_clique(r, np, clique);
        
        // 回溯
        r.erase(u);
    }
}
```





# 朴素算法优化

优化点在于，新引入了 **$S$ 集合** ，使用它可以对搜索过程进行剪枝，避免重复搜索，$R$ 和 $P$ 集合的含义未变。

- **$R$ 集合：** 保存当前团节点

- **$P$ 集合：** 当前团的邻接点集，该集合为空时，说明找到一个极大团

- **$S$ 集合：** 搜索到当前位置，已经搜索过的节点。
    - 递归时，搜索过的节点个数增加
    - 回溯时，搜索过的节点个数减少



按顺序遍历节点，以每个节点为团的第一个节点，找极大团。假设以节点 $1$ 开始极大团都已找到，接下来，找以节点 $2$ 开始的极大团：

- **$2$ 与 $1$ 相连，还需要找以 $1$ 为第二个点的团吗？不需要！！！包含 $1$ 的极大团都已经找到，没必要重复。**
- 假设团的第二个节点为 $3$，第三个节点为 $4$，....，当 $P$ 集合已为空时，找到了一个极大团 ${2, 3, 4, ...}$​
- 然后，需要回溯到 $4$，找其它团。弹出 $4$，压入 $5$ 作为第三个节点
- **$5$ 与 $4$ 相连，还需要找以 $4$ 为第四个点的团吗？不需要！！！以 $2$ 和 $3$ 节点为前提，包含 $4$ 的所有极大团都已经找到，没必要重复找了。**



$S$ 集合中包含的就是 $1$ 和 $4$ 这种已经搜索过，没必要再重复搜索的节点

- $1$ 这种节点：已经搜索过的祖辈节点。因为包含它的所有极大团都已经找到了。
- $4$ 这种节点：已经搜索过的兄弟节点。当回溯到 $3$ 时，应该将其从 $S$ 集合中移除，包含 $4$ 的所有极大团还没有找完。



# Bron Kerbosch v1

## 原理

Bron Kerbosch 是经典的团搜索算法，其中 $P$   和 $S$ 集合含义发生了 “略微变化”： 

```cpp
Bron-Kerbosch Algorithm(Version 1)
R={}   //当前团节点
P={v}  //当前团邻接点集中，未被搜索过的点。每当搜索完一个节点，需要从集合中移除
S={}   //当前团邻接点集中，已经被搜索过的节点。每当搜索完一个节点，需要加入到集合当中

BronKerbosch(R, P, S):
   if P and S are both empty:
       report R as a maximal clique
   for each vertex v in P:
       BronKerbosch1(R ⋃ {v}, P ⋂ N(v), S ⋂ N(v))
       P := P \ {v}
       S := S ⋃ {v}
```



### P 集合变化

**每当搜索完一个节点，直接将其从 $P$ 中移除** 这样做有什么用？用处大了！！！

- 朴素优化算法中，用 $S$ 集合来标记已访问过的节点，遇到时直接剪枝
- 何不直接将其扼杀在摇篮中？访问一个移除一个，连重复访问的机会都不给
- 太狠了，那 $S$ 集合是不已经没有存在的必要了？目前看着是，但接着往下看。



### S 集合变化

**$P$ 集合变化也带来了后遗症：无法通过是否为空来判断是否找到了极大团！** 为了解决这个问题，重新引入了 $S$ 集合，但含义却发生了变化：只表示当前层已访问过的节点。



`if P and S are both empty: report R as a maximal clique`：

- **$P$ 集合为空，$S$ 集合为空：** 此时才意味着找到了一个极大团。
- **$P$ 集合为空，$S$ 集合不为空：** 此时当前团并非极大团。找极大团找到一半时，发现还可以往当前团中加入新的节点，但它位于 $S$ 集合中，也就是被标记为已访问状态，需要剪枝，导致找到的是非极大团。



`BronKerbosch1(R ⋃ {v}, P ⋂ N(v), S ⋂ N(v))` 其中 `S ⋂ N(v)` 是干什么？

- 必须维护 $S$​ 集合的性质不发生变化：**当前团邻接点集中未搜索过的节点**
- 想要通过 **$P$ 集合为空，$S$ 集合为空** 来判断找到的是极大团，就必须这么做。



## 实现

```cpp
void bron_kerbosch_v1(set<int> &r, set<int> &p, set<int> &s, set<int> &clique, const vector<set<int>> &g) {
    if(p.empty() && s.empty()) {
        if(r.size() > clique.size()) {
            clique = r;
        }
        return;
    }

    while(!p.empty()) {
        int u = *p.begin();
        r.insert(u);

        set<int> np, ns;
        for(int v : g[u]) {
            if(p.find(v) != p.end()) {
                np.insert(v);
            }
            if(s.find(v) != s.end()) {
                ns.insert(v);
            }
        }

        bron_kerbosch(r, np, ns, clique, g);

        r.erase(u);
        p.erase(u);
        s.insert(u);
    }
}
```



# Bron Kerbosch v2

## 问题

![/assets/image/22.jpeg](/assets/image/22.jpeg)

从 `1` 开始搜索时，找到了极大团 `{1, 6}` 和 `{1, 2, 3, 4, 5}`。

- 从 `2` 开始搜索时，递归向下搜索到 `3, 4, 5`，最后却发现 `1` 已经搜索过了
- 从 `3` 开始搜索时，递归向下搜索到 `4, 5`，最后还是被 `1` 终结了
- ...
- 显然上述都是无效搜索



## 原理

 `Bron Kerbosch v2` 通过 **关键点优化** 来解决上述问题：**从邻接点集中选择一个点作为关键点，然后不再搜索与该点相邻的节点**。以上图为例：

- 选取 `1` 作为关键点，其搜索完毕之后，不再搜索 `2, 3, 4, 5, 6`，直接从 `7` 开始搜索
- 对于 `2, 3, 4, 5` 这种跳过，比较好理解，毕竟都在同一个极大团里面，无论怎么搜索都是无用功，直接跳过就行了
- 对于 `6` 这种跳过，会遗漏掉极大团 `{6, 7, 8}` ，不过没关系，从 `7` 开始搜索时会发现这个极大团的



```cpp
Bron-Kerbosch Algorithm(Version 2)
R={}   //当前团节点
P={v}  //当前团邻接点集中，未被搜索过的点。每当搜索完一个节点，需要从集合中移除
S={}   //当前团邻接点集中，已经被搜索过的节点。每当搜索完一个节点，需要加入到集合当中

BronKerbosch(R, P, S):
   if P and S are both empty:
       report R as a maximal clique
   choose a pivot vertex u in P ⋃ S
   for each vertex v in P \ N(u):   
       BronKerbosch1(R ⋃ {v}, P ⋂ N(v), S ⋂ N(v))
       P := P \ {v}
       S := S ⋃ {v}
```



本质上：剪枝了 $P$ 为空且 $S$​ 不为空的无效搜索，剪枝效果仅限于和关键点有关的部分，`{6, 7, 8}` 这些与关键点无关的团还是存在无效搜索问题。能否维护多个关键点？？？？？？？如果可以：

- 则可以彻底解决搜索到 “非极大团” 的无效搜索问题
- 解决了上述问题也就意味着 $P$ 为空时，找到的就是极大团，$S$​ 集合自然也就失去了存在的意义 
- 当前版本只会寻找一个关键点，因此 $S$​ 还是去不掉的。
- 理论上是可行的，应该有个 v3 终极版本
- 对 P 中所有节点先跑并查集，分成几类，就有几个关键点



## 实现

```cpp
void bron_kerbosch_v2(set<int> &r, set<int> &p, set<int> &s, set<int> &clique, vector<set<int>> &g) {
    if(p.empty() && s.empty()) {
        if(r.size() > clique.size()) {
            clique = r;
        }
        return;
    }

    // 关键点的所有邻居
    auto &nodes = g[*p.begin()];
    for(auto it = p.begin(); it != p.end();) {
        // 不管跳过还是不跳过，迭代器都需要跳到下一个
        int u = *it++;
        // 关键点的邻居节点，直接跳过
        // 跳过！！！而不需要删除
        if(nodes.find(u) != nodes.end()) continue;
        
        // 新节点加入当前团
        r.insert(u);

        // 计算新的 P 和 S 集合
        set<int> np, ns;
        for(int v : g[u]) {
            if(p.find(v) != p.end()) {
                np.insert(v);
            }
            if(s.find(v) != s.end()) {
                ns.insert(v);
            }
        }

        bron_kerbosch_v2(r, np, ns, clique, g);

        // 回溯
        r.erase(u);
        
        // 当前节点从 P 中移除，并加入到 S 中
        // 前面已经将迭代器自增了，不会涉及迭代器失效问题
        p.erase(u);
        s.insert(u);
    }
}

void solve() {
    int n = 0, m = 0;
    cin >> n >> m;
    vector<set<int>> g(n);
    for(int i = 0; i < m; ++i) {
        int u = 0, v = 0;
        cin >> u >> v;
        g[u].insert(v);
        g[v].insert(u);
    }
    set<int> s, p, r, clique;
    for(int i = 0; i < n; ++i) {
        p.insert(i);
    }
    bron_kerbosch_v2(s, p, r, clique, g);
}
```



# 练习

[POJ All Friends](http://poj.org/problem?id=2989)

```cpp
#include <cstdio>
#include <cstring>

using namespace std;
const int N = 130;
bool mp[N][N];

int r[N][N], p[N][N], s[N][N];
int n, m, ans;

void dfs(int d, int rn, int pn, int sn) {
    if(pn == 0 && sn == 0) {
        ++ans;
    }
    
    //pivot vertex（选邻居子多的节点更优）
    int u = p[d][0];
    for(int i = 0; i < pn; ++i) {
        int v = p[d][i];
        //对关键节点的邻居减枝
        if(mp[u][v]) continue;
        
        //计算下一次 r 集合
        for(int j = 0; j < rn; ++j) {
            r[d + 1][j] = r[d][j];
        }
        r[d + 1][rn] = v;
        
        //计算下一次 p 集合
        int pnn = 0;
        //为什么不能通过以下方式实现从 P 中移除已访问节点
        //for(int j = i; j < pn; ++j) {
        for(int j = 0; j < pn; ++j) {
            int x = p[d][j];
            if(mp[v][x]) {
                p[d + 1][pnn++] = x;
            }
        }
        
        //计算下一次 s 集合
        int snn = 0;
        for(int j = 0; j < sn; ++j) {
            int x = s[d][j];
            if(mp[v][x]) {
                s[d + 1][snn++] = x;
            }
        }
        dfs(d + 1, rn + 1, pnn, snn);
        
        //从 P 中移除已访问节点（0 和其它节点之间没有边）
        p[d][i] = 0;
        //将搜索过的节点加入 S 集合
        s[d][sn++] = v;
        if(ans > 1000) return;
    }
}

int work() {
    ans = 0;
    for(int i = 0; i < n; ++i) {
        p[1][i] = i + 1;
    }
    dfs(1, 0, n, 0);
    return ans;
}

int main() {
    while(~scanf("%d %d", &n, &m)) {
        memset(mp, 0, sizeof(mp));
        for(int i = 1; i <= m; ++i) {
            int u, v;
            scanf("%d %d", &u, &v);
            mp[u][v] = mp[v][u] = 1;
        }
        int tmp = work();
        if(tmp > 1000) puts("Too many maximal sets of friends.");
        else printf("%d\n", tmp);
    }
    return 0;
}
```

