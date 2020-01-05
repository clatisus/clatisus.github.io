---
title: 2020-1-5-Daily-Exercise
date: 2020-01-05 23:31:36
tags:
- 动态规划
- 双指针
- 二分法
---

## 完全平方数

给定正整数 `n`，找到若干个完全平方数（比如 `1, 4, 9, 16, ...`）使得它们的和等于 `n`。你需要让组成和的完全平方数的个数最少。

示例 1:
```
输入: n = 12
输出: 3 
解释: 12 = 4 + 4 + 4.
```

示例 2:
```
输入: n = 13
输出: 2
解释: 13 = 4 + 9.
```

### 动态规划

`dp[i]` 表示数字 `i` 最少可以由多少个完全平方数组成

初始化 `dp[i] = i` ，因为所有数都可以由 `i` 个 `1` 组成

$$
dp[i] = \min_{j < sqrt(i)} dp[i - j * j] + 1
$$

```
int numSquares(int n) {
    vector<int> dp(n + 1);
    for (int i = 1; i <= n; ++i) {
        dp[i] = i;
        for (int j = sqrt(i); j >= 1; --j) {
            dp[i] = min(dp[i], dp[i - j * j] + 1);
        }
    }
    return dp[n];
}
```

## 搜索二维矩阵 II

编写一个高效的算法来搜索 `m x n` 矩阵 `matrix` 中的一个目标值 `target`。该矩阵具有以下特性：

每行的元素从左到右升序排列。

每列的元素从上到下升序排列。

示例:

现有矩阵 matrix 如下：
```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```
给定 `target = 5`，返回 `true`。

给定 `target = 20`，返回 `false`。

### 二分法（TODO）

### 双指针

分析开始位置：
- 左上角，不可行
    - 往➡️，值增加
    - 往⬇️，值增加
- 右上角，可行
    - 往⬅️，值减少
    - 往⬇️，值增加
- 左下角，可行
    - 往➡️，值增加
    - 往⬆️，值减少
- 右下角，不可行
    - 往⬅️，值减少
    - 往⬆️，值减少

假设选择开始位置为左下角，则可以如下分析
1. 设矩阵左下角元素 `matrix[i][j]` ，它是第 `i` 行最小值，同时也是第 `j` 列最大值
2. 若 `target < matrix[i][j]` (小于第 `i` 行最小值)，则排除第 `i` 行，令 `i--`
3. 若 `target > matrix[i][j]` (大于第 `j` 列最大值)，则排除第 `j` 列，令 `j++`
4. 循环 2~3 直到找到 `target`，或所有行列均被排除

```
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    if (!matrix.size()) return false;
    if (!matrix[0].size()) return false;
    int i = matrix.size() - 1, j = 0;
    while (i >= 0 && j < matrix[0].size()) {
        if (matrix[i][j] > target) --i;
        else if (matrix[i][j] < target) ++j;
        else return true;
    }
    return false;
}
```

