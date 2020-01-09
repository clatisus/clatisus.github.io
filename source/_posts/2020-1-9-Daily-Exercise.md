---
title: 2020-1-9-Daily-Exercise
date: 2020-01-09 22:18:49
tags:
- 优先队列
- 二叉树
- 字符串
---

## K个离原点最近的点

我们有一个由平面上的点组成的列表 points。需要从中找出 K 个距离原点 (0, 0) 最近的点。

（这里，平面上两点之间的距离是欧几里德距离。）

你可以按任何顺序返回答案。除了点坐标的顺序之外，答案确保是唯一的。

示例 1：
```
输入：points = [[1,3],[-2,2]], K = 1
输出：[[-2,2]]
解释： 
(1, 3) 和原点之间的距离为 sqrt(10)，
(-2, 2) 和原点之间的距离为 sqrt(8)，
由于 sqrt(8) < sqrt(10)，(-2, 2) 离原点更近。
我们只需要距离原点最近的 K = 1 个点，所以答案就是 [[-2,2]]。
```

示例 2：
```
输入：points = [[3,3],[5,-1],[-2,4]], K = 2
输出：[[3,3],[-2,4]]
（答案 [[-2,4],[3,3]] 也会被接受。）
 ```

提示：
```
1 <= K <= points.length <= 10000
-10000 < points[i][0] < 10000
-10000 < points[i][1] < 10000
```

### 优先队列
优先队列默认是大根堆，如果要小根堆，可以存入负数。

```c++
vector<vector<int>> kClosest(vector<vector<int>>& points, int K) {
    priority_queue<pair<int, int>> q;
    for (int i = 0; i < points.size(); ++i) {
        int dis = points[i][0] * points[i][0] + points[i][1] * points[i][1];
        if (q.size() < K) {
            q.emplace(dis, i);
            continue;
        }
        if (q.top().first > dis) {
            q.pop();
            q.emplace(dis, i);
        }
    }
    vector<vector<int>> ans;
    while (!q.empty()) {
        ans.push_back(points[q.top().second]);
        q.pop();
    }
    return ans;
}
```

## 最大层内元素和

给你一个二叉树的根节点 root。设根节点位于二叉树的第 1 层，而根节点的子节点位于第 2 层，依此类推。

请你找出层内元素之和 最大 的那几层（可能只有一层）的层号，并返回其中 最小 的那个。

示例：
```
输入：[1,7,0,7,-8,null,null]
输出：2
解释：
第 1 层各元素之和为 1，
第 2 层各元素之和为 7 + 0 = 7，
第 3 层各元素之和为 7 + -8 = -1，
所以我们返回第 2 层的层号，它的层内元素之和最大。
```

提示：
```
树中的节点数介于 1 和 10^4 之间
-10^5 <= node.val <= 10^5
```

### BFS

```c++
int maxLevelSum(TreeNode* root) {
    queue<pair<TreeNode*, int>> q;
    q.emplace(root, 1);
    int ans = 0, last = 0, sum = INT_MIN, maxsum = INT_MIN;
    while (!q.empty()) {
        TreeNode* p = q.front().first;
        int layer = q.front().second;
        q.pop();
        if (layer != last) {
            ans = sum > maxsum ? last : ans;
            maxsum = sum > maxsum ? sum : maxsum;
            sum = 0;
            last = layer;
        } 
        sum += p->val;
        if (p->left != NULL)
            q.emplace(p->left, layer + 1);
        if (p->right != NULL)
            q.emplace(p->right, layer + 1);
    }
    return ans;
}
```

## 密钥格式化

给定一个密钥字符串S，只包含字母，数字以及 '-'（破折号）。N 个 '-' 将字符串分成了 N+1 组。给定一个数字 K，重新格式化字符串，除了第一个分组以外，每个分组要包含 K 个字符，第一个分组至少要包含 1 个字符。两个分组之间用 '-'（破折号）隔开，并且将所有的小写字母转换为大写字母。

给定非空字符串 S 和数字 K，按照上面描述的规则进行格式化。

示例 1：
```
输入：S = "5F3Z-2e-9-w", K = 4
输出："5F3Z-2E9W"
解释：字符串 S 被分成了两个部分，每部分 4 个字符；
     注意，两个额外的破折号需要删掉。
```

示例 2：
```
输入：S = "2-5g-3-J", K = 2
输出："2-5G-3J"
解释：字符串 S 被分成了 3 个部分，按照前面的规则描述，第一部分的字符可以少于给定的数量，其余部分皆为 2 个字符。
```

主要问题在如何处理特殊情况（刚好整除）

方法一：最后erase掉字符串首元素
`str.erase(pos, number)`：从 str[pos] 开始删掉 number 个元素
```c++
string licenseKeyFormatting(string S, int K) {
    string nodash;
    for (int i = 0; i < S.length(); ++i) {
        if (S[i] == '-') continue;
        nodash += S[i];
    }
    int first = nodash.length() % K;
    string ans;
    for (int i = 0; i < nodash.length(); ++i) {
        if (i < first) {
            ans += toupper(nodash[i]);
            continue;
        }
        ans += '-';
        for (int j = 0; j < K; ++j) {
            ans += toupper(nodash[i + j]);
        }
        i += K - 1;
    }
    if (!first && ans.size()) ans.erase(ans.begin());
    return ans;
}
```

方法二：如果余数为0，则把第一段当成K长度处理
```c++
string licenseKeyFormatting(string S, int K) {
    string nodash;
    for (int i = 0; i < S.length(); ++i) {
        if (S[i] == '-') continue;
        nodash += S[i];
    }
    int first = nodash.length() % K;
    string ans;
    if (first == 0) first = K;
    for (int i = 0; i < nodash.length(); ++i) {
        if (i < first) {
            ans += toupper(nodash[i]);
            continue;
        }
        ans += '-';
        for (int j = 0; j < K; ++j) {
            ans += toupper(nodash[i + j]);
        }
        i += K - 1;
    }
    return ans;
}
```

## 独特的电子邮件

每封电子邮件都由一个本地名称和一个域名组成，以 @ 符号分隔。

例如，在 alice@leetcode.com中， alice 是本地名称，而 leetcode.com 是域名。

除了小写字母，这些电子邮件还可能包含 '.' 或 '+'。

如果在电子邮件地址的本地名称部分中的某些字符之间添加句点（'.'），则发往那里的邮件将会转发到本地名称中没有点的同一地址。例如，"alice.z@leetcode.com” 和 “alicez@leetcode.com” 会转发到同一电子邮件地址。 （请注意，此规则不适用于域名。）

如果在本地名称中添加加号（'+'），则会忽略第一个加号后面的所有内容。这允许过滤某些电子邮件，例如 m.y+name@email.com 将转发到 my@email.com。 （同样，此规则不适用于域名。）

可以同时使用这两个规则。

给定电子邮件列表 emails，我们会向列表中的每个地址发送一封电子邮件。实际收到邮件的不同地址有多少？
 

示例：
```
输入：["test.email+alex@leetcode.com","test.e.mail+bob.cathy@leetcode.com","testemail+david@lee.tcode.com"]
输出：2
解释：实际收到邮件的是 "testemail@leetcode.com" 和 "testemail@lee.tcode.com"。
 ```

提示：
```
1 <= emails[i].length <= 100
1 <= emails.length <= 100
每封 emails[i] 都包含有且仅有一个 '@' 字符。
```

```c++
int numUniqueEmails(vector<string>& emails) {
    set<string> ans;
    for (int i = 0; i < emails.size(); ++i) {
        string s;
        int j = 0, flag = 0;
        for (j = 0; j < emails[i].length() && emails[i][j] != '@'; ++j) {
            if (emails[i][j] == '+') flag = 1;
            if (emails[i][j] == '.' || flag) continue;
            s += emails[i][j];
        }
        for (j; j < emails[i].length(); ++j) {
            s += emails[i][j];
        }
        ans.insert(s);
    }
    return ans.size();
}
```