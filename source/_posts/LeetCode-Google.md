---
title: LeetCode-Google
date: 2019-12-08 13:56:54
tags:
---
## Leetcode 面试合集

- ### 139 单词拆分

给定一个非空字符串 s 和一个包含非空单词列表的字典 wordDict，判定 s 是否可以被空格拆分为一个或多个在字典中出现的单词。

说明：

拆分时可以重复使用字典中的单词。
你可以假设字典中没有重复的单词。
示例 1：
```
输入: s = "leetcode", wordDict = ["leet", "code"]
输出: true
解释: 返回 true 因为 "leetcode" 可以被拆分成 "leet code"。
```
示例 2：
```
输入: s = "applepenapple", wordDict = ["apple", "pen"]
输出: true
解释: 返回 true 因为 "applepenapple" 可以被拆分成 "apple pen apple"。
     注意你可以重复使用字典中的单词。
```

示例 3：
```
输入: s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
输出: false
```

**动态规划**

`bool dp[s.length()]` 

`dp[i]`表示从`s[0]`开始的长度为`i`的字符串可以被分割到字典中

$$
dp[i] = \cup_{j<i,\ s[j,\ i)\ \in\ wordDict} dp[j] (dp[0]=true, )
$$

```c++
    bool wordBreak(string s, vector<string>& wordDict) {
        if (s.length() == 0) return false;
        if (wordDict.size() == 0) return false;
        bool dp[s.length() + 1] = {};
        dp[0] = true;
        for (int i = 1; i <= s.length(); ++i) {
            for (int j = 0; j < i; ++j) {
                // substr 里是 i - j 因为这里的i是dp的i，dp[i]对应的是s[i - 1]这个字符的情况
                if (dp[j] && (wordDict.end() != find(wordDict.begin(), wordDict.end(), s.substr(j, i - j))) ){
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.length()];
    }
```

- ### 131 分割回文串

给定一个字符串 s，将 s 分割成一些子串，使每个子串都是回文串。

返回 s 所有可能的分割方案。

示例:

```
输入: "aab"
输出:
[
  ["aa","b"],
  ["a","a","b"]
]
```

**动态规划**

与上一道题类似，都是考虑`i`结尾的前面的串，如果在`j`位置可以被分割成回文，且`j~i`的部分也是回文，那么拼在一起就是回文。当`j`和`i`符合上述规定，`dp[i]`就是在`dp[j]`的每一个分割方案最后加上`j~i`

- 求串的分割方案数 $dp[i] = \sum_{j<i,\ isPard(j,i)} dp[j]$ $(dp[0]=1)$ (不然就全都加起来是0)

- 求串最少可以被分为几个 $dp[i] = min_{j<i, isPard(j,i)}(dp[j] + 1)$ $(dp[0]= 0)$
- 求串最多可以被分为几个  $dp[i] = max_{j<i, isPard(j,i)}(dp[j] + 1)$ $(dp[0]= 0)$

```c++
    vector<vector<string>> partition(string s) {
        int n = s.length();
        vector<vector<vector<string>>> dp(n + 1);
        dp[0].push_back({});
        for (int i = 1; i <= n; ++i) {
            for (int j = 0; j < i; ++j) {
                if (isPard(s, j, i)) {
                    for (auto k : dp[j]) {
                        dp[i].push_back(k);
                        dp[i].back().push_back(s.substr(j, i - j));

                    }
                }
            }
        }
        return dp[n];
    }
    bool isPard(string s, int left, int right) {
        for (int i = left, j = right-1; i < j; ++i, --j) {
            if (s[i] != s[j]) return false;
        }
        return true;
    }
```

**递归**

递归的本质就是枚举，一层一层的枚举。depth表示当前深度

```
aacba
		    a						depth=1
	aa	         a a				depth=2
aac  aa c   a ac    a a c			depth=3
```

每一层只用考虑当前字符串，加上后一个字符串的情况：	

- 紧贴前一个字符串t，`tx`
- 如果前一个组合是回文，一个字符本身也是回文，所以可以分开放在后面`t, x`
  - 可以得知，如果前一个组合不是回文，那么再单独加一个字符也没有意义，因为前面已经不合法，所以可以少去一些重复遍历

从下一层回来的时候，要回溯之前的操作，删掉加入的`x`

```c++
    vector<vector<string>> ans;
    vector<string> tmp;
    int n;
    string str;
    vector<vector<string>> partition(string s) {
        n = s.size();
        str = s;
        tmp.push_back(s.substr(0, 1));
        dfs(1);
        return ans;
    }
    
    void dfs(int depth) {
        if (depth == n) {	//遍历到最深层，是回文就加入答案
            if (isPard(tmp.back(), 0, tmp.back().length())) {
                ans.push_back(tmp);
            }
            return;
        }
        tmp.back() += str.substr(depth, 1); //紧贴前一个
        dfs(depth + 1);
        tmp.back().pop_back(); //删掉最后一个添加上去的字符
        if (isPard(tmp.back(), 0, tmp.back().length())) {
            tmp.push_back(str.substr(depth, 1)); //加入单独的子串
            dfs(depth + 1);
            tmp.pop_back();	//删掉加入的子串
        }
        
    }
    
    bool isPard(string s, int left, int right) {
        for (int i = left, j = right-1; i < j; ++i, --j) {
            if (s[i] != s[j]) return false;
        }
        return true;
    }
```

- ### 152 最大连续乘积

给定一个整数数组 `nums` ，找出一个序列中乘积最大的连续子序列（该序列至少包含一个数）。

**示例 1:**

```
输入: [2,3,-2,4]
输出: 6
解释: 子数组 [2,3] 有最大乘积 6。
```

**示例 2:**

```
输入: [-2,0,-1]
输出: 0
解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。
```

**动态规划**

`dp[i]`表示以`i`结尾的最大连续乘积，$dp[i]=\begin{cases}
max(dp[i-1]_{max} * dp[i], dp[i]) & \text{dp[i] >= 0}\\
min(dp[i-1]_{min} * dp[i], dp[i])& \text{dp[i] < 0}
\end{cases} $

由于只和前一项有关，可以用两个变量来记录。

```c++
    int maxProduct(vector<int>& nums) {
        int mmax = INT_MIN, imax = 1, imin = 1;
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] < 0) {
                swap(imax, imin);
            }
            imax = max(imax*nums[i], nums[i]);
            imin = min(imin*nums[i], nums[i]);
            
            mmax = max(mmax, imax);
        }
        return mmax;
    }
```

- ### 238 除自身以外数组的乘积

给定长度为 *n* 的整数数组 `nums`，其中 *n* > 1，返回输出数组 `output` ，其中 `output[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积。

**示例:**

```
输入: [1,2,3,4]
输出: [24,12,8,6]
```

**说明:** 请**不要使用除法，**且在 O(*n*) 时间复杂度内完成此题。

**进阶：**
你可以在常数空间复杂度内完成这个题目吗？（ 出于对空间复杂度分析的目的，输出数组**不被视为**额外空间。）



用一个数组记录前缀乘积，然后再从右往左乘上后缀乘积

```c++
    vector<int> productExceptSelf(vector<int>& nums) {
        vector<int> ans(nums.size(), 1);
        ans[0] = 1;
        ans[1] = nums[0];
        for (int i = 2; i < nums.size(); ++i) {
            ans[i] = ans[i - 1]*nums[i - 1];
        }
        int right = nums[nums.size() - 1];
        for (int i = nums.size() - 2; i >= 0; --i) {
            ans[i] *= right;
            right *= nums[i];
        }
        return ans;
    }
```

- ### 179 最大数

给定一组非负整数，重新排列它们的顺序使之组成一个最大的整数。

**示例 1:**

```
输入: [10,2]
输出: 210
```

**示例 2:**

```
输入: [3,30,34,5,9]
输出: 9534330
```

**说明:** 输出结果可能非常大，所以你需要返回一个字符串而不是整数。

相当于按照新规则排序

> 假设（不是一般性），某一对整数 $a$ 和 $b$ ，我们的比较结果是 $a$ 应该在 $b$ 前面，这意味着 $a\frown b > b\frown a$，其中 $\frown$ 表示连接。如果排序结果是错的，说明存在一个 $c$ ， $b$ 在 $c$ 前面且 $c$ 在 $a$ 的前面。这产生了矛盾，因为 $a\frown b > b\frown a$ 和 $b\frown c > c\frown b$ 意味着 $a\frown c > c\frown a$ 。换言之，我们的自定义比较方法保证了传递性，所以这样子排序是对的。

```c++
    static bool cmp(string a, string b) {
        string s1 = a + b;
        string s2 = b + a;
        return s1 > s2;
    }
    string largestNumber(vector<int>& nums) {
        vector<string> s;
        for (int i : nums) {
            s.push_back(to_string(i));
        }
        sort(s.begin(), s.end(), cmp);
        if (s[0] == "0") return "0";
        string ans;
        for (string i : s) {
            ans += i;
        }
        return ans;
    }
    
```

- ### 寻找重复数

给定一个包含 *n* + 1 个整数的数组 *nums*，其数字都在 1 到 *n* 之间（包括 1 和 *n*），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。

**示例 1:**

```
输入: [1,3,4,2,2]
输出: 2
```

**示例 2:**

```
输入: [3,1,3,4,2]
输出: 3
```

**说明：**

1. **不能**更改原数组（假设数组是只读的）。
2. 只能使用额外的 *O*(1) 的空间。
3. 时间复杂度小于 *O*(*n*2) 。
4. 数组中只有一个重复的数字，但它可能不止重复出现一次。

**快慢指针**

重复数一定有循环

> 如果我们对 nums 进行这样的解释，即对于每对索引 i 和值 vi而言，“下一个” vj 位于索引 vi 我们可以将此问题减少到循环检测。
>
> 算法：
> 首先，我们可以很容易地证明问题的约束意味着必须存在一个循环。因为 nums 中的每个数字都在 11 和 nn 之间，所以它必须指向存在的索引。此外，由于 00 不能作为 nums 中的值出现，nums[0] 不能作为循环的一部分。

```c++
    int findDuplicate(vector<int>& nums) {
        int slow = nums[nums[0]];
        int fast = nums[slow];
        while (fast != slow) {
            slow = nums[slow];
            fast = nums[nums[fast]];
        }
        
        int ans = nums[0];
        while (ans != slow) {
            ans = nums[ans];
            slow = nums[slow];
        }       
        return ans;
    }
```

- ### 202 快乐数

编写一个算法来判断一个数是不是“快乐数”。

一个“快乐数”定义为：对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和，然后重复这个过程直到这个数变为 1，也可能是无限循环但始终变不到 1。如果可以变为 1，那么这个数就是快乐数。

**示例:** 

```
输入: 19
输出: true
解释: 
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1
```

**快慢指针**：快乐数则会进入111111的循环，所有不快乐数的数位平方和计算，最后都会进入 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 的循环中，所以快慢指针相等的时候，为1就是快乐的

```c++
    bool isHappy(int n) {
        int slow = n, fast = square(n);
        while (slow != fast) {
            slow = square(slow);
            fast = square(fast);            
            fast = square(fast);
        }
        return slow == 1;
    }
    int square(int n) {
        int ans = 0;
        while (n) {
            ans += ((n % 10) * (n % 10));
            n /= 10;
        }
        return ans;
    }
```

- ### 两整数之和

**不使用**运算符 `+` 和 `-` ，计算两整数 `a` 、`b` 之和。

**示例 1:**

```
输入: a = 1, b = 2
输出: 3
```

**示例 2:**

```
输入: a = -2, b = 3
输出: 1
```

**位运算**：需要进位的地方： a & b 为 1 的位。加法用异或坐，进位一直左移，直到全部进位完。

```c++
    int getSum(int a, int b) {
        while (b) {
            unsigned int carry = (unsigned int)(a & b);
            a = a ^ b;
            b = carry << 1;
        }
        return a;
    }
```

- ### 134 加油站

在一条环路上有 *N* 个加油站，其中第 *i* 个加油站有汽油 `gas[i]` 升。

你有一辆油箱容量无限的的汽车，从第 *i* 个加油站开往第 *i+1* 个加油站需要消耗汽油 `cost[i]` 升。你从其中的一个加油站出发，开始时油箱为空。

如果你可以绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1。

**说明:** 

- 如果题目有解，该答案即为唯一答案。
- 输入数组均为非空数组，且长度相同。
- 输入数组中的元素均为非负数。

**示例 1:**

```
输入: 
gas  = [1,2,3,4,5]
cost = [3,4,5,1,2]

输出: 3

解释:
从 3 号加油站(索引为 3 处)出发，可获得 4 升汽油。此时油箱有 = 0 + 4 = 4 升汽油
开往 4 号加油站，此时油箱有 4 - 1 + 5 = 8 升汽油
开往 0 号加油站，此时油箱有 8 - 2 + 1 = 7 升汽油
开往 1 号加油站，此时油箱有 7 - 3 + 2 = 6 升汽油
开往 2 号加油站，此时油箱有 6 - 4 + 3 = 5 升汽油
开往 3 号加油站，你需要消耗 5 升汽油，正好足够你返回到 3 号加油站。
因此，3 可为起始索引。
```

**示例 2:**

```
输入: 
gas  = [2,3,4]
cost = [3,4,3]

输出: -1

解释:
你不能从 0 号或 1 号加油站出发，因为没有足够的汽油可以让你行驶到下一个加油站。
我们从 2 号加油站出发，可以获得 4 升汽油。 此时油箱有 = 0 + 4 = 4 升汽油
开往 0 号加油站，此时油箱有 4 - 3 + 2 = 3 升汽油
开往 1 号加油站，此时油箱有 3 - 3 + 3 = 3 升汽油
你无法返回 2 号加油站，因为返程需要消耗 4 升汽油，但是你的油箱只有 3 升汽油。
因此，无论怎样，你都不可能绕环路行驶一周。
```



1. 初始化 `total` 和 `res` 为 0 ，并且选择 `0` 号加油站为起点。

2. 遍历所有的加油站：

   - 每一步中，都通过加上 `gas[i]` 和减去 `cost[i]` 来更新 `total` 和 `res` 。

   - 如果在 `i + 1` 号加油站， `res` < 0 ，将 `i + 1` 号加油站作为新的起点`ans`，同时重置 `res = 0` ，让油箱也清空。**从上一次重置的加油站到当前加油站的任意一个加油站出发，到达当前加油站之前， `res` 也一定会比 0 小**

3. 如果 `total < 0` ，返回 `-1` ，否则返回 `ans`


  ```c++
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int total = 0, res = 0, ans = 0;
        for (int i = 0; i < gas.size(); ++i) {
            total += (gas[i] - cost[i]);
            if (res < 0) {
                res = 0;
                ans = i;
            }
            res += (gas[i] - cost[i]);
        }
        return total >= 0 ? ans : -1;
    }
  ```

