$Prefix\ Array$ 和 $Suffix\ Array$ 道理是一样的，只不过一个是从前往后，一个是从后往前，本文只对 $Prefix\ Array$ 进行介绍，$Suffix\ Array$ 还请自行脑补。

先来看看两个例子。

# Prefix Sum Array

## 实现

```cpp
void func(const vector<int> &nums) {
    vector<int> prefix(nums.size());
    prefix[0] = nums[0];
    for(int i = 1; i < nums.size(); ++i) {
        prefix[i] = prefix[i - 1] + nums[i];
    }
    //...
}
```
## 分析

对于 $prefix$ 数组，我们不妨称呼它为 $nums$ 的 $prefix\ sum\ array$ ，它的本质是：

- 声明了一个与长度与 $nums$ 相同的数组；
- 其中每个元素，都等于 $nums$ 中它位置前面所有元素的 和，即：

$$
prefix[i] = sum(nums[0], nums[1], ..., nums[i])
$$



# Prefix Max Array

## 实现

```cpp
void func(const vector<int> &nums) {
    vector<int> prefix(nums.size());
    prefix[0] = nums[0];
    for(int i = 1; i < nums.size(); ++i) {
        prefix[i] = max(prefix[i - 1], nums[i]);
    }
    //...
}
```
## 分析

对于 $prefix$ 数组，我们不妨称呼它为 $nums$ 的 $prefix\ max\ array$ ，它的本质是：

- 声明了一个长度与 $nums$ 相同的数组；
- 其中每个元素，都等于 $nums$ 中它位置前面所有元素的 最大值，即：

$$
prefix[i]\ =\ max(nums[0],\ nums[1],\ ...,\ nums[i])
$$



# 抽象

## Prefix Array

对它们进行抽象，可以理解成：它们中的元素都是 原数组中前缀数组的某个函数！即：
$$
prefix[i] = f(nums[0], nums[1], ... nums[i])
$$

- 对于 $prefix\ sum\ array$ 而言，$f$ 函数的功能是 求和；
- 对于 $prefix\ max\ array$ 而言，$f$ 函数的功能是 求最大值；

$Prefix\ Array$ 就是 $prefix\ sum\ array$，$prefix\ max\ array$，$prefix\ min\ array$ 这些东西的总称。

## 更多示例

$Prefix\ Array$ 家族还有很多很多其它成员：

- $Prefix\ Count\ Array$: 前缀数组中 $tar$ 出现多少次

$$
prefix[i]\ =\ count(nums[0],\ nums[1],\ ...,\ nums[i],\ tar)
$$



- $Prefix\ Count\ Less Array$: 可以用 线段树 或 树状数组来实现

$$
prefix[i]\ = (nums[0] < nums[i]) + (nums[1] < nums[i]) + ... + (nums[i - 1] < nums[i])
$$





- $Prefix\ First\ Greater/Less\ Array$: 需要借助 单调栈 来实现

$$
prefix[i]\ = nums[i - 1],\ nums[i - 2],\ ...,\ nums[0] 中第一个比 nums[i] 大/小 的元素
$$



- $Prefix\ Max Of K Element\ Array$: 原数组中位置在其前面的 $K$ 个元素的最大值，与 $prefix\ max\ array$ 的不同在于，它把前面元素的个数限制为了 $K$ 个，而不是前面所有元素；需要借助 单调队列来实现 来实现

$$
prefix[i]\ =\ max(nums[i],\ nums[i - 1],\ ...,\ nums[i - k + 1])
$$



之所以介绍这么多的家族成员，并非是说这些成员有多重要，它们一点都不重要，名字都是乱起的，而是为了说明 $Prefix\ Array$ 这种思想应用的广泛性！



# 练习

说了这么多，这东西到底有啥用？当你做完下面练习，就会发现真的很好用：不仅是一个解题神器，更是一个性能优化神器，而且这玩意还跟动态规划有着千丝万缕的关系。
- [LeetCode 42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)(Prefix/Suffix Max Array);
- [LeetCode 1696. Jump Game VI](https://leetcode.com/problems/jump-game-vi/)(Prefix Max-Of-K-Element Array + dp)
