```cpp
//初始化
vector<vector<int>> dp(n, vector<int>(n, INT_MAX / 2));
for(int i = 0; i < n; ++i) {
    for(int j = 0; j < n; ++j) {
        //w[i][j] 为权重
        dp[i][j] = i == j ? 0 : w[i][j];
    }
}

//计算最短距离
for(int k = 0; k < n; ++k) {
    for(int i = 0; i < n; ++i) {
        for(int j = 0; j < n; ++j) {
            //默认值设为 INT_MAX / 2 防止这里溢出
            dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j]);
        }
    }
}
```

