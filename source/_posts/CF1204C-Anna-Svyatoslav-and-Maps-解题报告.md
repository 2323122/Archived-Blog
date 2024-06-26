---
title: CF1204C Anna, Svyatoslav and Maps 解题报告
categories: 解题报告
toc: true
date: 2019-08-25 15:22:02
tags:
	- Floyd
	- 栈
top:
thumbnail: https://i.loli.net/2019/08/25/TjyxX3ubsSrfle8.jpg
---

考场没做出来，身败名裂

# 题目链接

[(Luogu)CF1204C Anna, Svyatoslav and Maps](https://www.luogu.org/problem/CF1204C)

# 题意简述

给你一张图和一个序列，要求找出其最短的子序列，满足两个序列首尾元素相同，并且子序列中每个点都在原序列最短路径上（好像没说清楚

<!--more-->

# 解题思想

看到这个$n$很小，我们考虑`Floyd`先求出最短路。

然后对于原序列，我们**贪心**地考虑每三个点：
如果中间的点在另外两个点的**最短路径**上，我们就可以去掉它。

这个可以通过`Floyd`后得到的距离进行判断。

所以，我们需要维护一下这三个点：
对于每一个被去掉的点，显然不用考虑，然后当前用来更新的点肯定是完成的序列中最后一个被更新的，所以我们只需要维护答案队列的首。

考虑使用栈：
对于每一个原序列的元素，取出它与后一个元素与栈顶元素进行比较更新：
若答案符合上述要求，则将**中间元素**加入栈中，否则跳过。

需要注意的是，序列的`N`与图的`N`不是同一个，数组要开够。

# 参考代码

```c++
#include <cstdio>
#include <cctype>

namespace FastIO {
    inline int read() {
        char ch = getchar(); int r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
        return r * w;
    }
    void _write(int x) {
        if(x < 0) putchar('-'), x = -x;
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(int x) {
        _write(x);
        puts("");
    }
}

using namespace FastIO;

const int N = 110;
const int M = 1e6 + 6;

template <typename T> inline T min(T a, T b) {return a < b ? a : b;}

int n, m, a[M];
int f[N][N];
char mp[N][N];
int s[M], stop;

int main() {
    n = read();
    for(register int i = 1; i <= n; i++) {
        scanf("%s", mp[i] + 1);
    }
    for(register int i = 1; i <= n; i++) {
        for(register int j = 1; j <= n; j++) {
            if(mp[i][j] == '0') f[i][j] = 0x3f3f3f3f;
            else f[i][j] = 1;
        }
    }
    for(register int i = 1; i <= n; i++) {
        f[i][i] = 0;
    }
    m = read();
    for(register int i = 1; i <= m; i++) {
        a[i] = read();
    }
    for(register int k = 1; k <= n; k++) {
        for(register int i = 1; i <= n; i++) {
            for(register int j = 1; j <= n; j++) {
                f[i][j] = min(f[i][j], f[i][k] + f[k][j]);
            }
        }
    }
    s[++stop] = a[1];
    for(register int i = 2; i <= m; i++) {
        if(f[s[stop]][a[i + 1]] != f[s[stop]][a[i]] + f[a[i]][a[i + 1]]) s[++stop] = a[i];
    }
    printf("%d\n", stop);
    for(register int i = 1; i <= stop; i++) printf("%d ", s[i]);
    return 0;
}
```

