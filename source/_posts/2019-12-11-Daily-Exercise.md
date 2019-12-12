---
title: 2019-12-11-Daily-Exercise
date: 2019-12-11 01:35:13
tags: 字符串，strstr函数
---

## 12.11 字符串替换问题

输入一个字符串，用`grass`替换字符串中所有`orangesheep`（忽略大小写），其他部分按照输入的字符串输出

### 暴力

复杂度$O(mn)$，$m=strlen(orangesheep), n=输入字符串的长度$

```c++
#include <cstdio>
#include <cstring>
#include <ctype.h>

char str[110000], str_lower[110000];
const char old_str[] = "orangesheep";

int main() {
    int n;
    scanf("%d\n", &n);

    char c;
    int i = 0;
    for (i = 0; EOF != (c = getchar()); ++i) {
        str[i] = c;
        str_lower[i] = tolower(c);
    }
    str[i] = '\0';
    str_lower[i] = '\0';
    int size = i;
    for (i = 0; i < size; ++i) {
        int j = i;
        for (j = i; j - i < 11 && str_lower[j] == old_str[j - i]; ++j);
        if (j - i == 11) {
            printf("grass");
            i += 10;
            continue;
        }
        printf("%c", str[i]);
    }
    return 0;
}
```



### 使用strstr

复杂度$O(m+n)$，每次匹配得到地址之后，移动头指针，再匹配下一个

```c++
#include <cstdio>
#include <cstdlib>
#include <algorithm>
#include <vector>
#include <cstring>
#include <cmath>
#include <cassert>
#include <map>

const int MAX_S = 1200;

const char pattern[] = "orangesheep";
const char replace[] = "grass";
char s[MAX_S];
char lower_s[MAX_S];

int main() {
    int T;
    scanf("%d", &T);
    int tot = 0;
    while (T--) {
        fgets(s, MAX_S, stdin);
        int n = strlen(s);
        while (n && (s[n - 1] == '\n' || s[n] == '\r')) {
            s[n - 1] = '\0';
            --n;
        }
        if (n == 0) {
            ++T;
            continue;
        }
        int m = strlen(pattern);
        for (int i = 0; i < n; ++i) {
            lower_s[i] = (char) tolower(s[i]);
        }
        lower_s[n] = '\0';

        int i = 0;
        for (char *begin = strstr(lower_s + i, pattern); begin != NULL; begin = strstr(lower_s + i, pattern)) {
            int j = begin - lower_s;
            s[j] = '\0';
            printf("%s%s", s + i, replace);
            ++tot;
            i = j + m;
        }

        printf("%s\n", s + i);
    }
    printf("%d\n", tot);
    return 0;
}
```