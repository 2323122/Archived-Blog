---
title: Luogu P3199 [HNOI2009]最小圈 解题报告
categories: 解题报告
date: 2019-07-17 21:12:11
tags:
	- 实域二分
	- 图论
	- dfs
top:
---

是实域二分耶！

# 题目链接

[**P3199** [HNOI2009]最小圈](https://www.luogu.org/problemnew/show/P3199) 

# 题意简述

给你一张图，求每个环的边长平均值的最小值。

<!--more-->

# 解题思想

直接做不好做，看到极值考虑二分。

二分答案后，我们对每一条边减去这个答案，然后跑负环。如果有负环，说明答案可以更小，此题完结。

需要注意一下边权是浮点数，所以不管是答案还是`dist[]`数组都要开浮点型，不然会炸。

实域二分听说`for(次数)`比`while()  + eps `更精确？没试。

还有边数莫名开小...

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <cstring>

inline int read() {
    char ch = getchar(); int r = 0, w = 1;
    while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
    while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
    return r * w;
}

const int N = 6010;
const int M = N << 1;
const double eps = 1e-10;
const int INF = 1e7;

int n, m, cnt;
int head[N], nxt[M], ver[M];
double edge[M], d[N];
bool v[N];

inline void add(int x, int y, double z) {
    ver[++cnt] = y, edge[cnt] = z, nxt[cnt] = head[x], head[x] = cnt;
}

inline void init() {
    memset(v, 0, sizeof(v));
    memset(d, 0, sizeof(d));
}

bool dfs(int x, double ans) {
    v[x] = 1;
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i], z = edge[i];
        if(d[y] > d[x] + z - ans - eps) {
            if(v[y]) return 1;
            d[y] = d[x] + z - ans;
            if(dfs(y, ans)) return 1;
        }
    }
    v[x] = 0;
    return 0;
}

inline bool check(double ans) {
    init();
    for(register int i = 1; i <= n; i++) {
        if(v[i]) continue;
        if(dfs(i, ans)) return 1;
    }
    return 0;
}

int main() {
    n = read(), m = read();
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read();
        double z;
        scanf("%lf", &z);
        add(x, y, z);
    }
    double l = -INF, r = INF;
    while(r - l > eps) {
        double mid = (l + r) / 2;
        if(check(mid)) r = mid;
        else l = mid;
    }
    printf("%.8lf", r);
    return 0;
}
```

