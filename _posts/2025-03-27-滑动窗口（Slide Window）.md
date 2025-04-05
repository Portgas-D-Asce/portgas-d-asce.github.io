---
layout: page
tags: 数据结构与算法
image: /assets/image/4.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 滑动窗口（Slide Window）
---


窗口 == 子数组

窗口通常维护了某个性质，在滑动过程中，这个性质具有单调性，需要维护这个性质满足某个条件

最小窗口问题可以转化为 00001111 型二分查找，最大窗口问题可以转化成 11110000 型二分查找，但效率低


# 最大窗口

```cpp
// 窗口初始位置及长度
int idx = -1, len = 0;
for(int i = 0, j = 0; i < n; ++i) {
    // 将第 i 个元素添加到窗口里面
    c[s[i] - 'A']++;
    
    // 添加新元素到窗口后可能会使窗口条件遭到破坏
    while(!check(c)){
        // 不断地从窗口左侧移出元素，直至窗口条件重新成立
        c[s[j++] - 'A']--;
    }
    
    // 更新最大窗口
    if(i - j + 1 > len) {
        len = i - j + 1;
        idx = j;
    }
}
```



# 最小窗口

```cpp
// 窗口初始位置及长度
int idx = -1, len = INT_MAX;
for(int i = 0, j = 0; i < n; ++i) {
    // 将第 i 个元素加入窗口
    c[s[i] - 'A']++;
    
    // 新元素加入可能会导致窗口性质成立，但当前不一定是最小窗口
    while(check(c)) {
        // 更新最小窗口
        if(i - j + 1 < len) {
            len = i - j + 1;
            idx = j;
        }
        // 不断地从窗口左侧移出元素，直到性质不成立
        c[s[j++] - 'A']--;
    }
}
```

# 练习
