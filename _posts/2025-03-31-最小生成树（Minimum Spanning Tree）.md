---
layout: page
tags: 数据结构与算法 图论
image: /assets/image/25.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 最小生成树（Minimum Spanning Tree）
---

# 最小生成树
带权重的无向图 -> 连通分量 -> 生成树 -> 最小生成树

两种实现算法（都是基于贪心策略）：
- kruscal
- prim



# Kruskal

## 原理
$kruskal$ 是基于 **并查集** 来实现的：
- 先将所有边按权重排序
- 按权重从小到大遍历边
  - 如果边的两个顶点属于已经属于同一集合，该边不属于最小生成树；
  - 否则，该边属于最小生成树；
- 权重为负值也是没有问题的



## 实现
```cpp
int find(vector<int> &uf, int x) {
    return uf[x] == x ? x : (uf[x] = find(uf, uf[x]));
}

int mst(vector<vector<int>> &edges, int n) {
    int total = 0;
    vector<int> uf(n);
    for(int i = 0; i < n; ++i) {
        uf[i] = i;
    }

    sort(edges.begin(), edges.end());

    for(int i = 0; i < edges.size(); ++i) {
        int w = edges[i][0];
        int u = edges[i][1], pu = find(uf, u);
        int v = edges[i][2], pv = find(uf, v);
        if(pu == pv) continue;
        total += w;
        uf[px] = py;
    }
    return total;
}
```



## 复杂度

时间复杂度：$O(ElogV)$

空间复杂度：$O(V)$



# Prim

## 原理
$prim$ 和 $dijkstra$ 十分相似（有多相似，看下面实现就知道了），区别在于：
- **$dijkstra$ ：** 总是把 **开集合中距离起点最近的点加入到闭集合**；
- **$prim$ ：** 总是把 **开集合中距离已生成的最小生成树最近的点加入到闭集合**；

显然，要求权重为正



## 实现
```cpp
int mst(const vector<vector<pair<int, int>>> &g) {
    int n = g.size();
    vector<int> dist(n, INT_MAX);
    dist[0] = 0;
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({dist[0], 0});
    
    int total = 0;
    while(!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();
        if(d > dist[u]) continue;
        total += d;
        for(auto [v, w] : g[u]) {
            if(w >= dist[v]) continue;
            dist[v] = w;
            pq.push({dist[v], v});
        }
    }
    return total;
}
```


## 复杂度

时间复杂度：$O(ElogV)$

空间复杂度：$O(E)$



