---
title: 2020-1-8-Daily-Exercise
date: 2020-01-08 21:25:07
tags: 
- 动态规划
- 01背包
- 贪心
---

## Min Abs Difference of Server Loads
There are some processes that need to be executed. Amount of a load that process causes on a server that runs it, is being represented by a single integer. Total load caused on a server is the sum of the loads of all the processes that run on that server. You have at your disposal two servers, on which mentioned processes can be run. Your goal is to distribute given processes between those two servers in the way that, absolute difference of their loads will be minimized.

Given an array of n integers, of which represents loads caused by successive processes, return the minimum absolute difference of server loads.

Example 1:
```
Input: [1, 2, 3, 4, 5]
Output: 1
Explanation:
We can distribute the processes with loads [1, 2, 4] to the first server and [3, 5] to the second one,
so that their total loads will be 7 and 8, respectively, and the difference of their loads will be equal to 1.
```
把一个数组分成两个子数组，使得这两子数组和的差的绝对值最小。

### 动态规划
01背包，选择尽可能接近数组总和/2的选法

```c++
#include<cstdio>
#include<vector>
#include<algorithm>

int main() {
    std::vector<int> a = {1, 2, 3, 4, 5};
    int n = a.size();
    int sum = 0;
    for (int i = 0; i < n; ++i) {
        sum += a[i];
    }
    float size = sum / 2;

    std::vector<std::vector<int>> dp(n + 1);
    for (int i = 0; i < n + 1; ++i) {
        dp[i].resize(n + 1, 0);
    }
    for (int i = 1; i <= n; ++i) {
        for (int j = 0; j <= size; ++j) {
            dp[i][j] = dp[i - 1][j];
            if(j - a[i] >= 0) {
                dp[i][j] = std::max(dp[i][j], dp[i - 1][j - a[i]] + a[i]);
            }
        }
    }
    printf("%d\n", dp[n][size]);
    printf("%d\n", sum - dp[n][size]);
    return 0;
}
```

## 行相等的最少多米诺旋转
在一排多米诺骨牌中，`A[i]` 和 `B[i]` 分别代表第 `i` 个多米诺骨牌的上半部分和下半部分。（一个多米诺是两个从 1 到 6 的数字同列平铺形成的 —— 该平铺的每一半上都有一个数字。）

我们可以旋转第 `i` 张多米诺，使得 `A[i]` 和 `B[i]` 的值交换。

返回能使 `A` 中所有值或者 `B` 中所有值都相同的最小旋转次数。

如果无法做到，返回 -1.

 

示例 1：
```
输入：A = [2,1,2,4,2,2], B = [5,2,6,2,3,2]
输出：2
解释：
如果我们旋转第二个和第四个多米诺骨牌，我们可以使上面一行中的每个值都等于 2。
```

示例 2：
```
输入：A = [3,5,1,2,3], B = [3,6,3,3,4]
输出：-1
解释：
在这种情况下，不可能旋转多米诺牌使一行的值相等。
```

提示：

```
1 <= A[i], B[i] <= 6
2 <= A.length == B.length <= 20000
```

### 贪心

```c++
int minDominoRotations(vector<int>& A, vector<int>& B) {
    int n = A.size();
    int flagA = 0, a0 = A[0];
    int flagB = 0, b0 = B[0];
    for (int i = 0; i < n; ++i) {
        if ((a0 != A[i]) && (a0 != B[i])) {
            flagA = 1;
        }
        if ((b0 != A[i]) && (b0 != B[i])) {
            flagB = 1;
        }
        if (flagA && flagB) return -1;
    }
    int cntA = 0, cntB = 0, target = a0;
    if (flagA) {
        target = b0;
    }
    for (int i = 0; i < n; ++i) {
        if (A[i] == target && B[i] == target) {
            continue;
        } else if (A[i] == target) {
            cntA += 1;
        } else {
            cntB += 1;
        }
    }
    return min(cntA, cntB);
}
```

## Largest Subarray Length K
Array X is greater than array Y if the first non-matching element in both arrays has greater value in X than in Y

Given an array A contains N distinct integers and an integer K. Find the largest contiguous subarray length K.

$1\le K \le N \le 100$
$1\le A[j] \le 1000$ 

### 贪心

```c++
#include<cstdio>
#include<vector>

int main() {
    std::vector<int> a = {1, 2, 4, 3, 5};
    int k = 4;
    int ans = 0;
    for (int i = 0; i < a.size() - k + 1; ++i) {
        ans = a[ans] > a[i] ? ans : i;
    }
    for (int i = 0; i < k; ++i) {
        printf("%d%c", a[ans + i], i == k-1 ? '\n' : ' ');
    }
    return 0;
}
```