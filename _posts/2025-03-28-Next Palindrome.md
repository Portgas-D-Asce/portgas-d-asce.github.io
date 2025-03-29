---
layout: page
tags: 数据结构与算法 数论
image: /assets/image/24.jpeg
description: xxxxxxxxxxxxxxxxxx
author: pk
title: Next Palindrome
---

# 非严格

小于等于 or 大于等于当前数的最大回文

```cpp
string pre_palindrome(const string &s) {
    string t = s;
    int n = t.size();
    for(int i = 0, j = n - 1; i < j; ++i, --j) {
        t[j] = t[i];
    }
    // 可以通过 “将右半边变为左半边” 的方式得到
    if(t <= s) return t;
    
    // 否则 “先将左半边减 1，然后赋给右半边”
    for(int i = (n - 1) / 2; i >= 0; --i) {
        if(t[i] != '0') {
            t[n - 1 - i] = --t[i];
            break;
        }
        t[i] = t[n - 1 - i] = '9';
    }
    // “减一导致了位数变化” 10000 这种情况
    if(t[0] == '0') {
        t[0] = '9';
        t.pop_back();
    }
    // "0" 和 "1" 在前面筛选过了，不会出现在这里
    return t;
}
```



```cpp
string next_palindrome(const string &s) {
    string t = s;
    int n = t.size();
    for(int i = 0, j = n - 1; i < j; ++i, --j) {
        t[j] = t[i];
    }
    // 可以通过 “将右半边变为左半边” 的方式得到
    if(s <= t) return t;
    
    // 否则 “先将左半边加一，然后赋给右半边”
    for(int i = (n - 1) / 2; i >= 0; --i) {
        if(t[i] != '9') {
            t[n - 1 - i] = ++t[i];
            break;
        }
        t[i] = t[n - 1 - i] = '0';
    }
    // “加一导致了位数变化” 9999 这种情况
    if(t[0] == '0') {
        t[0] = '1';
        t.push_back('1');
    }
    return t;
}
```



# 严格

如何返回严格小于 or 大于当前数的最大回文？先将当前数减一，然后再找小于等于的最大回文不失为一个好的方法，但上面实现完美兼容严格小于版本，仅需稍加改造：

- 当输入值本身就是一个回文时，是不能直接返回的
- 小于 1 的最大回文是 0 而非空
- 小于 0 的最大回文该如何定义？？？

```cpp
string pre_palindrome(const string &s) {
    // 不特判 "0" 的时候会返回 "9"，返回 "" 吧，报错总比返回错误结果好
    if(s == "0") return "";
    // "1" 的时候会得到空字符串
    if(s =="1") return "0";
    string t = s;
    int n = t.size();
    for(int i = 0, j = n - 1; i < j; ++i, --j) {
        t[j] = t[i];
    }
    // 可以通过 “将右半边变为左半边” 的方式得到
    // 这里仅仅把等号去掉
    if(t < s) return t;
    
    // 否则 “先将左半边减 1，然后赋给右半边”
    for(int i = (n - 1) / 2; i >= 0; --i) {
        if(t[i] != '0') {
            t[n - 1 - i] = --t[i];
            break;
        }
        t[i] = t[n - 1 - i] = '9';
    }
    // “减一导致了位数变化” 10000 这种情况
    if(t[0] == '0') {
        t[0] = '9';
        t.pop_back();
    }
    return t;
}
```



```cpp
string next_palindrome(const string &s) {
    string t = s;
    int n = t.size();
    for(int i = 0, j = n - 1; i < j; ++i, --j) {
        t[j] = t[i];
    }
    // 可以通过 “将右半边变为左半边” 的方式得到
    // 这里仅仅把等号去掉
    if(s < t) return t;
    
    // 否则 “先将左半边加一，然后赋给右半边”
    for(int i = (n - 1) / 2; i >= 0; --i) {
        if(t[i] != '9') {
            t[n - 1 - i] = ++t[i];
            break;
        }
        t[i] = t[n - 1 - i] = '0';
    }
    // “加一导致了位数变化” 9999 这种情况
    if(t[0] == '0') {
        t[0] = '1';
        t.push_back('1');
    }
    return t;
}
```



# 扩展

重串：s = t + t 则 s 是一个重串，如何查找 next repeat ?



# 练习

[LeetCode 564. 寻找最近的回文数](https://leetcode.cn/problems/find-the-closest-palindrome/)

[LeetCode 2967. 使数组成为等数数组的最小代价](https://leetcode.cn/problems/minimum-cost-to-make-array-equalindromic/)

[479. 最大回文数乘积](https://leetcode.cn/problems/largest-palindrome-product/)



