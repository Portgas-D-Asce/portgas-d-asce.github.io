# 石头碰撞

若干堆石头两两相互碰撞，一样大则同时消失，否则留下剩余的石头

## 最后一个石头最小



## 最后一个石头最大





# 容斥原理



两个集合


$$
A \cup B = A + B - A \cap B
$$
三个集合
$$
A \cup B \cup C = A + B + C - A \cap B - A \cap C - B \cap C + A \cap B \cap C
$$
多个集合
$$
A_1 \cup A_2 \cup ... \cup A_n = \sum_{i = 1} ^{2^n} (-1)^{c_i} \cap{s_i}
$$
$\cap s_i$ 表示所有集合的子集，$c_i$ 表示子集中集合的个数，$n$ 个集合有 $2^n - 1$ 



# 位运算

__builtin_popcount

__builtin_popcountl

__builtin_popcountll





# 01 矩阵

有 m 个数，每个数都是一个 n 位的 01 字符串。

## 最小个数

异或和 == $2^n - 1$ 时，需要最少字符串的个数。

## 方案数

异或和 == $2^n - 1$​  的方案数



# xxx

SOS DP

FMT



# 前缀和

## 前缀和

### 二维容斥原理

$$
dp[i][j] = dp[i - 1][j] + dp[i][j - 1] - dp[i - 1][j - 1] + a[i][j]
$$

### 二维逐维度

```cpp
for(int i = 1; i <= n; i++)
    for(int j = 1; j <= n; j++)
        a[i][j] += a[i - 1][j];
for(int i = 1; i <= n; i++)
    for(int j = 1; j <= n; j++)
        a[i][j] += a[i][j - 1];
```

### 三维逐维度

```cpp
for(int i = 0; i <= n; ++i) {
    for(int j = 0; j <= n; ++j) {
        for(int k = 0; k <= n; ++k) {
            a[i][j][k] += a[i - 1][j][k];
        }
    }
}

for(int i = 0; i <= n; ++i) {
    for(int j = 0; j <= n; ++j) {
        for(int k = 0; k <= n; ++k) {
            a[i][j][k] += a[i][j - 1][k];
        }
    }
}

for(int i = 0; i <= n; ++i) {
    for(int j = 0; j <= n; ++j) {
        for(int k = 0; k <= n; ++k) {
            a[i][j][k] += a[i][j][k - 1];
        }
    }
}
```



## 高维前缀和





# FMT





# 命题

原命题为：若a，则b；

逆命题为：若b，则a；

否命题为：若非a，则非b；

逆否命题为：若非b，则非a； 

原命题和逆否命题为等价命题．如果原命题成立，逆否命题成立。

逆命题和否命题为等价命题，如果逆命题成立，否命题成立





# 余数性质



设 $y_1 = k_1x + b_1$，$y_2 = k_2x + b_2$，则：
$$
y_1 \times y_2 \% x = b_1 * b_2\ \%\ x \\
(y_1 + y_2) \% x = (b_1 + b_2) \% x
$$





```cpp
op[2][i] = op[2][i - 1] + op[2][i - 1] + op[3][i - 1];

```



