---
layout: page
tags: 字符串
image: /assets/image/53.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: 马拉车算法（Longest Palindromic Substring）
categories: [计算机, 数据结构与算法]
---

# Brute Force

可以先找到所有子串，然后判断每个子串是否为回文，期间记录最长回文子串即可：
- 一个长度为 $n$ 的字符串的子串的个数为： $n(n + 1) / 2$ ；
- 判断字符串是否为回文时间复杂度为 $O(n)$;

因此，总的时间复杂度为 $O(n^3)$ 。



# Better Than Brute Force

以第 $i$ 个字符为中心，向两边扩展（奇数、偶数都扩展一次），即可得到：

- 以第 $i$ 个字符为中心的最长回文（时间复杂度为 $O(n)$）
- 对每个字符进行同样的操作，总共需执行 $n$ 次
- 期间记录下最长回文即可。

显然，总的时间复杂度为 $O(n^2)$ 。



# 马拉车算法



## 算法步骤

可分为三步：
- 对原字符串 $s$ 预处理，得到预处理字符串 $t$ ，**使回文子串长度只能是奇数**，简化后续处理
- 计算 $t$ 的最长回文子串
- 还原出 $s$ 的最长回文子串



## 预处理

在 **Better Than Brute Force** 方案中我们需要考虑回文子串长度为奇数（eg，bcb）、偶数（eg，bccb）的两种情况



马拉车的预处理方法：在每个字符左右两侧都加上特殊字符，巧妙地避开了这个问题：
- 原字符串："acbcd";
- 预处理后字符串："#a#c#b#c#d#";

通过这样的预处理，使得回文子串长度只能为奇数，完全不必讨论偶数情况。



额外加一个特殊字符

- 可以预防左边界越界 "#a#c#b#c#a#"（整个串都是回文） 变为 "##a#c#b#c#a#"
- 为什么右边界不用预防？看不起 '\0' ？



## 预处理后字符串 t 的最长回文子串



### 变量说明

首先需要理解两个重要而又晦涩难懂的辅助变量：
- `mx`：当前已找到的回文子串能延伸到的最右端位置的 **下一个位置**
- `id`：延伸到最右端的位置的回文子串的中心点位置（预处理字符串回文子串长度必为奇数，所以一定有中心点位置）



其它一些变量及其含义：
- `p[id]`：以索引 `id` 为中心的最长回文子串的半径（包括中心点）；
- `i`：当前访问字符的索引；
- `j`：`i` 关于 `id` 的对称位置（即，`j = 2 * id - i`）；



### 核心代码

马拉车算法中最核心的一行代码

```cpp
p[i] = mx > i ? min(p[2 * id - i], mx - i) : 1;
```


当 $mx <= i$ 时：$i$ 处于 $mx$ 右端，没有可用信息，只能确定以 $i$ 为中心的最长回文子串的半径至少为1



当 $mx > i$ 时：$i$ 在 $mx$ 延伸范围内，存在可用信息：

- 当 $mx - i >= p[j]$ 时，根据对称，以 $j$ 为中心的最长回文子串 和 以 $i$ 为中心的最长回文子串都被包含在以 $id$ 为中心的最长回文子串内，如下图所示：

![/assets/content/17.png](/assets/content/17.png)





- 当 $mx - i < p[j]$ 时，根据对称，我们可以确定以$i$ 为中心的最长回文子串的半径至少为 $mx - i$，到底是多少还要继续进行比较，如下图所示：

![/assets/content/18.png](/assets/content/18.png)



## 原字符串 s 的最长回文子串

### 长度计算

先观察以下两种情况：
- $t$ 的最长回文子串 "#b#c#b#" 长度为 7，半径为 4，其所对应的 $s$ 的最长回文子串 "bcb" 长度为 3
- $t$ 的最长回文子串 "#b#c#c#b#"  长度为 9，半径为 5，其所对应的 $s$ 的最长回文子串 "bccb" 长度为 4



`s 最长回文子串长度 = t 最长回文子串半径长度 - 1`



### 起始索引计算

知道了 $s$ 最长回文子串的长度，只要再知道其在原字符串中的起始索引，就可以截取到最长回文子串。示例：



$s$ = "cambcbdn" （最长回文子串为奇数情况）：
- 预处理后：$t$ = "##c#a#m#b#c#b#d#n#"
- $t$ 最长回文子串：#b#c#b#，半径为4，其中心字符 'c' 的索引为10
- $s$ 最长回文子串：bcb，起始索引为 (10 - 4) / 2 = 3



$s$ = "ambccbdn"（最长回文子串为偶数情况）：
- 预处理后为：$t$ = "##a#m#b#c#c#b#d#n#"
- $t$ 最长回文子串：#b#c#c#b#，半径为5，其中心字符为'#'索引为9
- $s$ 最长回文子串：bccb，起始索引为 (9 - 5) / 2 = 2



`s 最长回文子串起始索引 =（t 最长回文子串中心字符索引 - t 最长回文子串半径）/ 2`



## 代码实现

```cpp
#include <string>
#include <vector>


string process(string s){
    string res = "##";
    for(int i = 0; i < s.size(); ++i) {
        res.push_back(s[i]);
        res.push_back('#');
    }
    return res;
}

string manacher(string s){
    string t = process(s);
    vector<int> p(t.size());
    int id = 0, mx = 0;
    int c = 0, r = 0;
    // 无法从 0 开始，p[i] 至少为 1，从 0 开始 i - p[i] = -1
    for(int i = 1; i < t.size(); ++i) {
        // 延伸内部存在可用信息
        p[i] = mx > i ? min(p[2 * id - i], mx - i) : 1;
        
        // 延伸外部不存在可用信息，需逐一比较
        while(t[i + p[i]] == t[i - p[i]]) {
            ++p[i];
        }
        
        // 更新延伸
        if(mx < i + p[i]) {
            mx = i + p[i];
            id = i;
        }
        
        // 记录 t 最长回文子串的半径及中心
        if(p[i] > r) {
            r = p[i];
            c = i;
        }
    }
    
    // 还原得到 s 的最长回文子串
    return s.substr((c - r) / 2, r - 1);
}
```


## 时间复杂度

时间复杂度：$O(n)$

空间复杂度：$O(n)$



# 封装

```cpp
template<typename T>
pair<int, int> manacher(const T& s, U start, U end){
    using U = typename type_traits<T>::value_type;
    int n = s.size();
    
    T t(2, start);
    for(auto ch : s) {
        t.push_back(ch);
        t.push_back(start);
    }
    t.push_back(end);
    
    int m = t.size();
    vector<int> p(m);
    int c = 0, r = 0;
    for(int i = 1, id = 0, mx = 0; i < m - 1; ++i) {
        // 延伸内部存在可用信息
        p[i] = mx > i ? min(p[2 * id - i], mx - i) : 1;
        
        // 延伸外部不存在可用信息，需逐一比较
        while(t[i + p[i]] == t[i - p[i]]) {
            ++p[i];
        }
        
        // 更新延伸
        if(mx < i + p[i]) {
            mx = i + p[i];
            id = i;
        }
        
        // 记录 t 最长回文子串的半径及中心
        if(p[i] > r) {
            r = p[i];
            c = i;
        }
    }
    
    // 还原得到 s 的最长回文子串, 起始位置 + 长度
    return {(c - r) / 2, r - 1};
}
```





# 练习题

[LeetCode 5. Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)
[LeetCode 647. Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings/)

[B. Non-Palindromic Substring](https://codeforces.com/contest/1943/problem/B)

