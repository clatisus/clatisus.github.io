---
title: Algorithm-Homework-2
date: 2019-12-08 13:52:33
tags:
---
### 二进制串转换

对于二进制串 a，b 而言，两个二进制串相比较，数字相同的位置一定不需要做变换，这样才能使得开销最小。

其次我们考虑交换这一操作，如果交换的两个数字位置不相邻，开销一定大于等于两个位置分别翻转，所以对于交换这一操作，我们只考虑相邻位置的交换，使得开销最小。

最后则是翻转操作，不满足上述两种情况的，都需要翻转处理。

$dp[i]$ 表示对于前 $i$ 个数字，使得 $a[1]$ 到 $a[i]$ 等于 $b[1]$  到 $b[i]$ 的最小开销是 $dp[i]$。

对于 $dp[i]$ 的状态转移，首先分为 $a[i] \neq b[i]$。

​	若 $a[i] == b[i - 1] \&\& a[i - 1] == b[i]$  ，则满足了相邻两个数字可以交换的条件，考虑应当交换还是翻转
$$
dp[i] = \min(dp[i - 2] + 1, dp[i - 1] + 1);
$$
​	若不满足上述条件，仅仅是 $a[i] \neq b[i]$，则只可能翻转
$$
dp[i] = dp[i - 1] + 1
$$
若 $a[i] == b[i]$，则
$$
dp[i] = dp[i - 1]
$$
代码如下

时间复杂度为 O(n)

```c++
#include <vector>
#include <cstdio>
#include <cstring>
#include <algorithm>

int main() {
    char a[100], b[100];
    scanf("%s", a + 1);
    scanf("%s", b + 1);
    int n = strlen(a + 1);
    std::vector<int> dp(n + 1);
    dp[0] = 0;
    dp[1] = a[1] == b[1] ? 0 : 1;
    for (int i = 2; i <= n; ++i) {
        if (a[i] != b[i]) {
            if (a[i - 1] == b[i] && a[i] == b[i - 1]) {
                dp[i] = std::min(dp[i - 2] + 1, dp[i - 1] + 1);
            } else {
                dp[i] = dp[i - 1] + 1;
            }
        } else {
            dp[i] = dp[i - 1];
        }
    }
    printf("%d", dp[n]);
}
```



### 最长递增子序列

$dp[i]$ 表示以 $i$ 结尾的最长递增子序列。

对于 $dp[i]$ 的确定，在 $j < i$ 和 $a[j] < a[i]$ 的情况下，找到最大的$dp[j] + 1$
$$
dp[i] = \max \limits_{j < i \\ a[j] < a[i]}(dp[j] + 1)
$$
代码如下

时间复杂度为 O(n^2)

```c++
#include <cstdio>
#include <algorithm>
#include <vector>

int solve(std::vector<int> &a) {
    int n = a.size();
    std::vector<int> dp(n + 1);
    int ans = 0;
    for (int i = 0; i < n; ++i) {
        int pre = 0;
        for (int j = 0; j < i; ++j) {
            if (a[j] < a[i]) {
                pre = std::max(dp[j], pre);
            }
        }
        dp[i] = pre + 1;
        ans = std::max(ans, dp[i]);
    }
    return ans;
}

int main() {
    std::vector<int> a = {5, 24, 8, 17, 12 ,25};
    printf("%d\n", solve(a));
    return 0;
}
```



### 括号匹配问题

假设字符串数组为 $s$，分两种情况考虑该问题：

1. $s[i], s[j]$ 不管能不能配对，都可以将$s[i,j]$分成两段字符串，求得和最大值
2. $s[i], s[j]$ 能配对，则考虑是 $s[i+1,j-1]$ 中合法的值加上头尾两个能配对的值，即 $+ 2$ ，与之前算出来的和最大值再比较取最大

$$
\left\{
\begin{array}{**l**}
dp[i][j] = \max(dp[i][k] + dp[k+1][j]), & \text{i}\le \text{k<j}\\
\ \ \\
dp[i][j]=\max(dp[i][j],dp[i+1][j-1]+2), &\text{s[i],s[j] is pair}
\end{array}
\right .
$$



```c++
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <cstring>
#include <vector>

int main() { 
    char str[100];
    scanf("%s", str + 1);
    int n = strlen(str + 1);
    std::vector<std::vector<int>> dp(n + 1);
    for (int i = 0; i <= n; ++i) {
        dp[i].resize(n + 1);
    }
    for (int len = 2; len <= n; ++len) {
        for (int i = 1; i <= n - len + 1; ++i) {
            int j = i + len - 1;
            if ((str[i] == '[' && str[j] == ']') ||
                (str[i] == '(' && str[j] == ')')) {
                dp[i][j] = std::max(dp[i][j], dp[i + 1][j - 1] + 2);
                }
            for (int k = i; k < j; ++k) {
                dp[i][j] = std::max(dp[i][j], dp[i][k] + dp[k + 1][j]);
            }
            
        }
    }
    printf("%d\n", dp[1][n]);
    return 0;
}
```



### 分组可行性判定

$dp[i] = \text{true | false}$ 表示前 $i$ 个元素是否可以按条件分组。

若 $dp[j]$  为 $\text{true}$，即表示前 $j$ 个可以分组，且此时 $a[i] - a[j + 1] \le d$，即满足题意，且 $i - j \ge k$ ，即这一组内至少有 $k$ 个，则 $dp[i]$ 也满足题意分组
$$
dp[i] = \mathop \cup \limits_{i-j \ge k \\ a[i] - a[j+1] \le d} dp[j]
$$


```c++
#include <cstdio>
#include <vector>

int d, k;

int main() { 
     // from a[1] to a[n], a[0] = 0 means a[0] is nothing
    std::vector<int> a = {0, 1, 1, 2, 3, 3, 4, 5, 10, 10, 19};
    scanf("%d%d", &d, &k);
    int n = a.size() - 1;
    std::vector<bool> dp(n + 1);
    dp[0] = true;
    for (int i = 1; i <= n; ++i) {
        for (int j = i - k; j >= 0; --j) {
            if (a[i] - a[j + 1] > d) {
                break;
            }
            if (dp[j]) {
                dp[i] = true;
                break;
            }
        }
    }
    puts(dp[n] ? "YES" : "NO");
    return 0;
}
```



### 最大分值问题

$sum[i]$ 表示前$i$个元素的和，$dp[i][part]$ 表示前 $i$ 个元素被分为 $part$ 段的最大结果
$$
dp[i][part] = \max_{j < i\\part\le k} (dp[j][part - 1] + (sum[i] - sum[j]) \% p)
$$

```c++
#include <cmath>
#include <cstdio>
#include <vector>

int main() {
    std::vector<int> a = {0, 3, 4, 7, 2};
    int n = a.size() - 1;
    int k, p;
    scanf("%d%d", &k, &p);
    std::vector<std::vector<int>> dp(n + 1);
    std::vector<int> sum(n + 1);
    for (int i = 0; i <= n; ++i) {
        dp[i].resize(k + 1);
    }
    for (int i = 1; i <= n; ++i) {
        sum[i] = sum[i - 1] + a[i];
        dp[i][1] = sum[i] % p;
    }
    for (int part = 2; part <= k; ++part) {
        for (int i = 1; i <= n; ++i) {
            for (int j = 1; j < i; ++j) {
                dp[i][part] = std::max(dp[i][part], dp[j][part - 1] + (sum[i] - sum[j]) % p);
            }
        }
    }
    printf("%d\n", dp[n][k]);
}
```

