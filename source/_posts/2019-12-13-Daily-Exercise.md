---
title: 2019-12-13-Daily-Exercise
date: 2019-12-13 01:58:52
tags: 动态规划
---

## 12.13 正则表达式匹配

### 动态规划

$dp[i][j]$ 表示 $s[i:]$ 和 $p[j:]$ 能成功匹配，从尾部开始递归。

初始化使得 $dp[m][n]$ 为true，开始dp

状态转移过程为：

- 如果 $p[j+1]$ 为 $*$ 
  - $dp[i][j] = dp[i][j + 2]$：$*$ 不起作用，字母重复0次
  - $dp[i][j] = match\ \&\ dp[i + 1][j]$：$*$起作用，则如果$s[i+1:]$ 与 $p[j:]$ 可以匹配，那么 $dp[i][j]$ 一定可以匹配
    - 有可能是 $*$ 前的字符出现一次导致匹配
    - 有可能是 $*$ 前的字符在 $dp[i + 1][j]$ 的时候已经匹配，此时再加匹配一次
- 否则，$dp[i][j] = match\ \&\ dp[i + 1][j + 1]$：如果$s[i+1:]$ 与 $p[j+1:]$ 可以匹配，那么 $dp[i][j]$ 一定可以匹配

时间复杂度 $O(mn)$

```c++
bool isMatch(string s, string p) {
    int m = s.length(), n = p.length();
    vector<vector<bool>> dp (m + 1);
    for (int i = 0; i < m + 1; ++i) {
        dp[i].resize(n + 1);
    }
    dp[m][n] = true;
    for (int i = m; i >= 0; --i) {
        for (int j = n - 1; j >= 0; --j) {
            bool match = (i < m && (s[i] == p[j] || p[j] == '.'));
            if (j + 1 < n && p[j + 1] == '*') {
                dp[i][j] = dp[i][j + 2] || (match && dp[i + 1][j]);
            } else {
                dp[i][j] = match && dp[i + 1][j + 1];
            }
        }
    }
    return dp[0][0];
}
```

