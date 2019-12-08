---
title: Algorithm-Homework-3
date: 2019-12-07 00:59:52
tags:
---
1. 假设 $01$ 串中，有 $m$ 段 $0$ 。

   - 如果 $m=1$，那么无论如何交换，这段 $0$ 的长度也不会变得更长。

   - 如果 $m\geq 2$ ，那么至少可以把一段 $0$ 中的某一个 $0$，交换到另一段 $0$ 的旁边，使其长度加 $1$。

       - 如果 $m = 2$，且有两段 $0$ 中间只有一个 $1$，可以经过交换把两段 $0$ 合并成一段 $0$。

       - 如果 $m > 2$，且有两段 $0$ 中间只有一个 $1$ ，可以把一个不在这两段中的 $0$，与这两段中的 $1$ 交换，可以获得额外的 $1$ 的收益。

   算法如下：

   - 先用 $\mathcal{O}(n)$ 的复杂度，找出每一段连续的 $0$ 所在的区间，我们设一共有 $m$ 段。并且同时用每一段的长度更新答案。
   - 若 $m\geq 2$，如上所述，我们一定可以将另外一段的一个 $0$ 交换至当前最优解的旁边，使得答案加 $1$。
   - 然后遍历所有段，如果存在两段中间只有一个 $1$，则可以用上述方法将两段 $0$ 中间的 $1$ 交换成 $0$，使得两段拼接起来，我们选择收益更高的方式即可。
   
   时间复杂度为 $\mathcal{O}(n)$。

```c++
#include <cstdio>
#include <vector>

int solve(std::vector<int> s) {
    std::vector<std::pair<int, int>> part;
    int ans = 0;
    for (int i = 0, j; i < s.size(); i = j) {
        for (j = i + 1; j < s.size() && s[i] == s[j]; ++j);
        if (!s[i]) {
            part.emplace_back(i, j - 1);
            ans = std::max(ans, j - i);
        }
    }
    int npart = part.size();
    if (npart >= 2) ans++;
    for (int i = 0; i < npart - 1; ++i) {
        if (part[i + 1].first - part[i].second != 2) continue;

        int len1 = part[i].second - part[i].first + 1;
        int len2 = part[i + 1].second - part[i + 1].first + 1;
        ans = std::max(ans, len1 + len2 + (npart > 2));
    }
    return ans;
}

int main() {
    std::vector<int> a = {1, 0, 0};
    std::vector<int> b = {0, 0, 1, 1, 0, 0, 0};
    std::vector<int> c = {1, 0, 0, 1, 1, 0, 0, 0, 1, 0};
    std::vector<int> d = {1, 1, 1, 1, 1, 1, 1};
    printf("%d %d %d %d\n", solve(a), solve(b), solve(c), solve(d));
}
```

2. 如果到目前为止的收益加起来 $\geq U$，我们考虑现在是否升级：

   - 升级我们会获得 $x_2\times \text{剩余天数（含今天）}$ 的收益，会付出 $U$ 的代价
   
   显然，如果我们要选择升级，那么越早升级越好（代价相同，收益随时间变大而递减）。所以只要收益大于代价，我们就贪心的升级即可。
   
   时间复杂度为 $\mathcal{O}(n)$。

```c++
#include <cstdio>

int solve(int x1, int x2, int u, int c, int n) {
    int ans = c, every = x1;
    for (int i = 1; i <= n; ++i) {
        if (ans >= u) {
            if (x2 * (n - i) > u) {
                ans -= u;
                every += x2;
            }
        }
        ans += every;
    }
    return ans;
}

int main() {
    printf("%d\n", solve(1, 2, 3, 4, 5));
}
```

3. 我们设每个桶最短的板长度依次为 $V_1, \dots, V_n$，不妨假定他们已经按升序排好序。题目给的条件 $\forall 1\leq x, y\leq n, \mid V_x-V_y\mid \leq l$，等价于 $V_n-V_1\leq l$。

   不妨假定 $a_1, \dots, a_m$ 也按升序排好序，那么显然有 $V_1=a_1$，且 $V_n\geq a_n$。容易证明 $V_n=a_n$ 可以取到，所以我们只需要判断是否有 $a_n-a_1\leq l$ 即可。我们可以用 `nth_element` 来找到第 $1$ 和第 $n$ 小的元素。

   时间复杂度为 $\mathcal{O}(m=n\times k)$。

```c++
#include <cstdio>
#include <vector>
#include <algorithm>

int m, n, k, l;
std::vector<int> a;

bool solve() {
    std::nth_element(a.begin(), a.begin() + n - 1, a.end());
    std::nth_element(a.begin(), a.begin(), a.end() + n);
    return (a[n - 1] - a[0] <= l);
}

int main() {
    scanf("%d%d%d", &n, &k, &l);
    m = n * k;
    for (int i = 0; i < m; ++i) {
        int tmp;
        scanf("%d", &tmp);
        a.push_back(tmp);
    }
    printf("%d\n", (int)solve());
    return 0;
}
```

4. $n$ 条边 $n$ 个点的连通无向图，这是一棵基环树。基环树上有一个环，环上每个点都「挂」着一棵树。我们只需要将环上的边按一个方向定向。然后每棵「挂」在环上的树上的每个非根节点，将它连父亲的那条边指向父亲即可。

   具体算法如下：
   
- 我们拓扑排序，将所有度为 $1$ 的点放入队列。每次取出队首的节点 $u$，枚举它的每条边 $(u,v)$，那么 $v$ 一定是它在树上的父亲节点，我们定向 $u\rightarrow v$，然后我们删掉这条边，将 $v$ 节点的度数减 $1$。若节点 $v$ 的度数为 $1$，那么我们将其放入队列。
   - 我们重复上述步骤，直到队列清空。那么此时，我们剩下的度数为 $2$ 的节点，就是环上的节点。
   - 我们从任意一个度数为 $2$ 的节点开始，遍历这个环，按照访问顺序给边定向即可。
   
   所有点和边都只遍历 $1$ 次，时间复杂度为 $\mathcal{O}(n)$。

```c++
#include <vector>
#include <cstdlib>
#include <queue>

const int MAX_N = 100010;

int n;
int du[MAX_N];
std::vector<int> vec[MAX_N];
std::vector<std::pair<int, int>> ans;

int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; ++i) {
        int u, v;
        scanf("%d%d", &u, &v);
        vec[u].push_back(v);
        vec[v].push_back(u);
        ++du[u];
        ++du[v];
    }

    int circle_node = n;
    std::queue<int> queue;
    for (int i = 1; i <= n; ++i) {
        if (du[i] == 1) {
            queue.push(i);
        }
    }
    while (!queue.empty()) {
        --circle_node;
        int u = queue.front();
        queue.pop();
        for (auto v : vec[u]) {
            // u -> v
            ans.emplace_back(u, v);
            if (--du[v] == 1) {
                queue.push(v);
            }
        }
    }

    for (int i = 1; i <= n; ++i) {
        if (du[i] == 2) {
            // find a node in circle
            int u = i;
            for (int j = 0; j < circle_node - 1; ++j) {
                --du[u];
                for (auto v : vec[u]) {
                    if (du[v] == 2) {
                        ans.emplace_back(u, v);
                        u = v;
                        break;
                    }
                }
            }
            ans.emplace_back(u, i);
            break;
        }
    }

    for (auto &p : ans) {
        printf("%d -> %d\n", p.first, p.second);
    }
    return 0;
}
```



5. 设 $\text{dis}(i, j)$ 表示点 $i$ 和点 $j$ 之间的曼哈顿距离。那么有 $\text{dis}(i, j) = \mid x_i-x_j\mid + \mid y_i - y_j\mid$。

   - 如果不经过传送点，那么显然直接从 $i$ 到 $j$ 是最优的。否则存在点 $k$，有 $\text{dis}(i, k) + \text{dis}(k, j) < \text{dis}(i, j)$，这显然是恒假的。所以从 $i$ 到 $j$ 的最短路就是 $\text{dis}(i, j)$。
   - 如果要经过传送点，那么一定是点 $i$ 到距离其最近的传送点进入，从离点 $j$ 最近的传送点出来，然后到点 $j$。

   时间复杂度 $\mathcal{O}(n)$。

```c++
#include <cstdio>
#include <vector>

int n;
std::vector<std::pair<int, int>> dot;
std::vector<int> t;

int solve(std::pair<int, int>x, std::pair<int, int>y) {
    int ansx = std::abs(y.first - x.first), ansy = std::abs(y.second - x.second);
    int ans = ansx + ansy;
    ansx = INT_MAX;
    ansy = INT_MAX;
    for (int i = 0; i < n; ++i) {
        if (!t[i]) continue;
        ansx = std::min(ansx, std::abs(dot[i].first - x.first) + std::abs(dot[i].second - x.second));
        ansy = std::min(ansy, std::abs(dot[i].first - y.first) + std::abs(dot[i].second - y.second));
    }
    if (ansx != INT_MAX) {
        ans = std::min(ans, ansx + ansy);
    }
    return ans;
}

int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; ++i) {
        int tmp1, tmp2, tmp3;
        scanf("%d%d", &tmp1, &tmp2);
        dot.emplace_back(tmp1, tmp2);
        scanf("%d", &tmp3);
        t.emplace_back(tmp3);
    }
    int x, y;
    scanf("%d%d", &x, &y);
    int ans = solve(dot[x], dot[y]);
    printf("%d\n", ans);
    return 0;
}
```



