---
title: Luogu P2680 运输计划 解题报告
categories: 解题报告
toc: true
date: 2019-10-14 20:58:07
tags:
	- 图论
	- 贪心
	- 树上差分
	- LCA
	- 树链剖分
	- 二分
top:
thumbnail: https://i.loli.net/2019/10/14/GaRjq9tWxdK713f.jpg
---

~~板子题套板子题套板子题 = 联赛题~~

# 题目链接

[P2680 运输计划](https://www.luogu.org/problem/P2680)

# 题意简述

给你一棵树，有边权，指定 $n$ 条链，可以将树上一条边的边权置为 $0$，求最大链长的最小值。

<!--more-->

# 解题思想

首先题目要求最大的最小，考虑二分答案。二分出这个值后，我们遍历一下待处理的每一条链，求出**原始链长**比二分的答案要大的链（下文中简称为**答案链**）。显然，我们要删去的边一定属于答案链的**并**。

考虑**树上差分**求出每一条边被答案链**覆盖的次数**，那么，被覆盖次数等于答案链数的边则是答案链的并。可以贪心地在答案链的并中取出**边权最大的**将其边权置为 $0$，然后进行判定：若置为 $0$ 后，答案链中**最长**的链比**二分的答案**要大，则二分出的答案过小。否则二分的答案过大。

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

const int N = 3e5 + 6;
const int M = N << 1;

int n, m;
int head[N], ver[M], nxt[M], edge[M], cnt;
int d[N], dis[N], length[N], fa[N], tp[N], sz[N], son[N];
int f[N], num, maxlen, maxdis;
int l, r;

inline int max(int a, int b) {return a > b ? a : b;}
inline void swap(int &a, int &b) {int c = a; a = b, b = c;}

inline void add(int x, int y, int z) {
    ver[++cnt] = y, edge[cnt] = z, nxt[cnt] = head[x], head[x] = cnt;
    ver[++cnt] = x, edge[cnt] = z, nxt[cnt] = head[y], head[y] = cnt;
}

void dfs1(int x) {
    sz[x] = 1;
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i], z = edge[i];
        if(y == fa[x]) continue;
        fa[y] = x;
        length[y] = z;
        d[y] = d[x] + 1;
        dis[y] = dis[x] + z;
        dfs1(y);
        sz[x] += sz[y];
        if(sz[son[x]] < sz[y]) son[x] = y;
    }
}

void dfs2(int x, int top) {
    tp[x] = top;
    if(son[x]) dfs2(son[x], top);
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(y == fa[x] || y == son[x]) continue;
        dfs2(y, y);
    }
}

int lca(int x, int y) {
    while(tp[x] != tp[y]) {
        if(d[tp[x]] > d[tp[y]]) swap(x, y);
        y = fa[tp[y]];
    }
    return d[x] < d[y] ? x : y;
}

void dfs(int x) {
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(y == fa[x]) continue;
        dfs(y);
        f[x] += f[y];
    }
    if(f[x] >= num && length[x] >= maxlen) maxlen = length[x];
}

struct Node {
    int x, y, LCA, len;
    Node() {}
    Node(int _x, int _y) : x(_x), y(_y), LCA(lca(_x, _y)), len(dis[x] + dis[y] - 2 * dis[LCA]) {}
} plan[N];

bool check(int maxtime) {
    memset(f, 0, sizeof f);
    num = 0, maxlen = 0, maxdis = 0;
    for(register int i = 1; i <= m; i++) {
        if(plan[i].len > maxtime) {
            int x = plan[i].x, y = plan[i].y, LCA = plan[i].LCA;
            f[x]++, f[y]++, f[LCA] -= 2;
            maxdis = max(maxdis, plan[i].len);
            num++;
        }
    }
    dfs(1);
    return maxdis - maxlen <= maxtime;
}

int main() {
    n = read(), m = read();
    for(register int i = 1; i < n; i++) {
        int x = read(), y = read(), z = read();
        add(x, y, z);
    }
    dfs1(1);
    dfs2(1, 1);
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read();
        plan[i] = Node(x, y);
        r = max(r, plan[i].len);
    }
    while(l < r) {
        int mid = (l + r) >> 1;
        if(check(mid)) r = mid;
        else l = mid + 1;
    }
    printf("%d\n", l);
    return 0;
}
```

