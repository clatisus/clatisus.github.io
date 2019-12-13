---
title: 2019-12-14-Daily-Exercise
date: 2019-12-14 14:53:23
tags:
- 动态规划
- 栈
---

给定一个只包含 `'('` 和 `')'` 的字符串，找出最长的包含有效括号的子串的长度。

示例 1:
```
输入: "(()"
输出: 2
解释: 最长有效括号子串为 "()"
```

示例 2:
```
输入: ")()())"
输出: 4
解释: 最长有效括号子串为 "()()"
```

### 动态规划

$dp[i]$ 表示以 $s[i]$ 结尾的最长有效括号字串

- $s[i - 1] = '('\ \&\ s[i] = ')'$ 说明这个字符串是 $".....()"$，$dp[i]=d[i - 2] + 2$
- $s[i - 1] = ')'\ \&\ s[i] = ')'$  说明这个字符串是 $".....))"$
  - 如果 $s[i - 1 - dp[i - 1]] = '('$ ，$dp[i] = dp[i - 1] + 2 + dp[i - 2 - dp[i - 1]]$

时间复杂度$O(n)$


```c++
int longestValidParentheses(string s) {
    int n = s.length();
    if (!n) return 0;
    vector<int> dp(n + 1, 0);
    for (int i = 1; i < n; ++i) {
        if (s[i - 1] == '(' && s[i] == ')') {
            if (i >= 2) {
                dp[i] = dp[i - 2] + 2;
            } else {
                dp[i] = 2;
            }
        } 
        if (i - 1 - dp[i - 1] >= 0 && s[i - 1] == ')' && s[i] == ')' && s[i - 1 - dp[i - 1]] == '(') {
            dp[i] = dp[i - 1] + 2;
            if (i - 2 - dp[i - 1] >= 0) {
                dp[i] += dp[i - 2 - dp[i - 1]];
            }
        }
    }
    int ans = 0;
    for (int i = 0; i < n; ++i) {
        ans = max(ans, dp[i]);
    }
    return ans;
}
```

### 栈

依次压入栈中，如果满足现在读到的 $s[i]$ 是 $)$，并且栈顶是 $($，就出栈，否则压栈

这样到最后只会剩下无法匹配好的括号，然后寻找到 头，两个无法匹配的括号之间，尾，这几者中间隔最长的一段，就是答案。

时间复杂度$O(n)$

```c++
int longestValidParentheses(string s) {
    stack<pair<char, int>> t;
    for (int i = 0; i < s.length(); ++i) {
        if (s[i] == ')' && !t.empty() && t.top().first == '(') {
            t.pop();
            continue;
        } 
        t.push(pair<char, int>(s[i], i));
    }
    if (t.empty()) return s.length();
    int ans = 0, last = s.length();
    while (!t.empty()) {
        int now = t.top().second;
        ans = max(ans, last - now - 1);
        last = now;
        t.pop();
    }
    ans = max(ans, last);
    return ans;
}
```