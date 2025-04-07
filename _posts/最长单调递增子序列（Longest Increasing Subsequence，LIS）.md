# 原理

贪心策略：

- 从左到右遍历
- 对于每个长度的单调递增子序列，总保持最后一个元素的值最小
- 经过这样处理，可以使后面插入更多的元素



# 实现

```cpp
//贪心：temp[i] 中存储着长度为 i + 1 的单调递增子序列最后一个元素的最小值
int lengthOfLIS(vector<int>& nums) {
    vector<int> temp;
    for(int num : nums) {
        //非严格单调递增 num >= temp.back()
        if(temp.empty() || num > temp.back()) {
            temp.push_back(num);
        } else {
            //非严格单调递增 upper_bound
            auto it = lower_bound(temp.begin(), temp.end(), num);
            *it = num;
        }
    }
    return temp.size();
}
```



# 思考

如何还原任一最长单调递增子序列？



# 练习

[LeetCode 300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

[LeetCode 354. 俄罗斯套娃信封问题](https://leetcode.cn/problems/russian-doll-envelopes/)