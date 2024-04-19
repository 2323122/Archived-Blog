---
title: 'Luogu P3436 [POI2006]PRO-Professor Szu 解题报告'
categories: 解题报告
toc: true
date: 2019-09-07 17:23:44
tags:
top:
thumbnail: https://i.loli.net/2019/09/09/pjdmJY6gXQzx1kS.jpg
---

输出不规范，彩笔两行泪。

# 题目链接

[P3436 [POI2006]PRO-Professor Szu](https://www.luogu.org/problem/P3436)

# 题意简述

给你一张图，多源一汇，求源点到汇点的最大路径方案数，并找出满足这些方案数的源点。

<!--more-->

# 解题思想

多源一汇，先考虑建反图。

考虑缩点，当原图存在包含点数大于 $1$ 的强连通分量时，方案有无数个。

当存在自环时，方案也有无数个。

其他情况原图是**一个或多个** $DAG$ 。可以找出包含汇点的那个 $DAG$，然后拓扑时计数，输出答案即可。

注意题目中给出的点数是源点的点数。

# 参考代码

```c++
#include <cstdio>
#include <cstring>
#include <cctype>

inline int read() {
    char ch = getchar(); int r = 0, w = 1;
    while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
    while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
    return r * w;
}

const int N = 1e6 + 6;
const int M = N << 2;
const int INF = 36501;

int n, m, ans;
int head[N], nxt[M], ver[M], dp[N], cnt;
int Head[N], Nxt[M], Ver[M], in[N], Cnt;
int dfn[N], low[N], scc[N], num[N], idx, tot;
int s[N], stop;
int q[N << 1], l, r;
bool v[N];

inline int min(int a, int b) {return a < b ? a : b;}
inline int max(int a, int b) {return a > b ? a : b;}

inline void add(int x, int y) {
    ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
}

inline void Add(int x, int y) {
    Ver[++Cnt] = y, Nxt[Cnt] = Head[x], Head[x] = Cnt;
    in[y]++;
}

void tarjan(int x) {
    dfn[x] = low[x] = ++idx;
    v[x] = 1;
    s[++stop] = x;
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(!dfn[y]) {
            tarjan(y);
            low[x] = min(low[x], low[y]);
        }
        else if(v[y]) {
            low[x] = min(low[x], dfn[y]);
        }
    }
    if(dfn[x] == low[x]) {
        tot++;
        while(stop) {
            int y = s[stop--];
            v[y] = 0;
            scc[y] = tot;
            num[tot]++;
            if(x == y) break;
        }
    }
}

inline void rebuild() {
    memset(v, 0, sizeof(v));
    cnt = 0;
    for(register int x = 1; x <= n; x++) {
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i];
            if(x == y) dp[scc[x]] = INF;
            if(scc[x] != scc[y]) Add(scc[x], scc[y]);
        }
    }
}

inline void TopSort() {
    dp[scc[n]] = 1;
    v[scc[n]] = 1;
    for(register int i = 1; i <= tot; i++) {
        if(!in[i]) q[++r] = i;
    }
    while(l < r) {
        int x = q[++l];
        for(register int i = Head[x]; i; i = Nxt[i]) {
            int y = Ver[i];
            if(v[x]) v[y] = 1;
            in[y]--;
            if(!in[y]) q[++r] = y;
            if(num[y] > 1 && v[y]) dp[y] = INF;
            dp[y] = min(dp[y] + dp[x], INF);
        }
    }
}

int main() {
    n = read() + 1, m = read();
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read();
        add(y, x);
    }
    for(register int i = 1; i <= n; i++) {
        if(!dfn[i]) tarjan(i);
    }
    rebuild();
    TopSort();
    for(register int i = 1; i <= n; i++) {
        ans = max(ans, dp[scc[i]]);
    }
    printf(ans == INF ? "zawsze\n" : "%d\n", ans);
    for(register int i = 1; i <= n; i++) {
        if(v[scc[i]] && dp[scc[i]] == ans) cnt++;
    }
    printf("%d\n", cnt);
    for(register int i = 1; i < n; i++) {
        if(v[scc[i]] && dp[scc[i]] == ans) printf("%d ", i);
    }
    return 0;
}
```

