---
title: Luogu P1967 货车运输 解题报告
categories: 解题报告
date: 2019-04-29 15:26:09
tags:
	- 图论
	- LCA
	- MST
top: 
---

卡空间的~~屑~~题

# 题目链接

[P1967 货车运输](<https://www.luogu.org/problemnew/show/P1967>)

# 题意简述

给你一个图，有边权，对于任意一组询问，输出给定两点间所有路径中边权的最小值的最大值。（即题意中**限重**的最大值）

<!--more-->

# 解题思想

~~跟二分没关系啊~~，考虑构造图的最大生成树，这样可以满足**最大值**这一条件。

此时，两点间的路径**有且仅有**一条：

$$E(x,y) = E(x, lca(x,y)) + E(lca(x, y), y)$$

在**树上倍增**的时候~~口胡~~**倍增优化DP**维护$min\{W(x, lca(x,y))\}$和$min\{W(lca(x,y)), y)\}$, 答案取$min$就行了。

需要注意的是查询的两点**不一定联通**，所以我们在`Kruskal`构造最大生成树的同时可以维护**图上两点的连通性**，查询时加入判断就行了。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <algorithm>

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
using namespace std;

const int N = 100010;
const int M = 400010;

inline void fileread() {
    freopen("a.in", "r", stdin);
    freopen("a.out","w",stdout);
}

struct Edge {
    int x, y, z;
}e[N];

int fa[N];
int head[N], edge[M], ver[M], nxt[M], cnt;
int lg[N], f[N][22], w[N][22], d[N];
int n, m, q;
bool v[N];

inline void add(int x, int y, int z) {
    ver[++cnt] = y, edge[cnt] = z, nxt[cnt] = head[x], head[x] = cnt;
}

inline bool cmp(Edge a, Edge b) {
    return a.z > b.z;
}

inline void init() {
    for(register int i = 1; i <= n; i++) {
        fa[i] = i;
        lg[i] = lg[i - 1] + (1 << lg[i - 1] == i);
        d[i] = 1;
    }
}

int find(int x) {return fa[x] = fa[x] == x ? x : find(fa[x]);}

inline bool merge(int x, int y) {
    int xx = find(x), yy = find(y);
    if(xx == yy) return 0;
    fa[xx] = yy;
    return 1;
}

inline void rebuild() {
    sort(e + 1, e + m + 1, cmp);
    int tot = 0;
    init();
    for(register int i = 1; i <= m; i++) {
        int x = e[i].x, y = e[i].y, z = e[i].z;
        if(merge(x, y)) {
            tot++;
            add(x, y, z);
            add(y, x, z);
            if(tot == n - 1) return;
        }
    }
}

void dfs(int x) {
    v[x] = 1;
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i], z = edge[i];
        if(!v[y]) {
            d[y] = d[x] + 1;
            f[y][0] = x;
            w[y][0] = z;
            dfs(y);
        }
    }
}

inline int lca(int x, int y) {
    if(find(x) != find(y)) return -1;
    int ans = 0x3f3f3f3f;
    if(d[y] > d[x]) swap(x, y);
    for(register int i=20; i>=0; i--) {
        if(d[f[x][i]]>=d[y]){
            ans = min(ans, w[x][i]);
            x = f[x][i]; 
        }
    }
    if(x == y) return ans;
    for(register int i = lg[d[x]]; i >= 0; i--) {
        if(f[x][i] != f[y][i]) {
            ans = min(ans, min(w[x][i], w[y][i]));
            x = f[x][i], y = f[y][i];
        }
    }
    ans = min(ans, min(w[x][0], w[y][0]));
    return ans;
}

int main() {
    n = read(), m = read();
    for(register int i = 1; i <= m; i++) {
        e[i].x = read(), e[i].y = read(), e[i].z = read();
    }
    rebuild();
    for(register int i = 1; i <= n; i++) {
        if(!v[i]) {
            d[i] = 1;
            dfs(i);
            f[i][0] = i;
            w[i][0] = 0x3f3f3f3f;
        }
    }
    for(register int i = 1; i <= 20; i++) {
        for(register int j = 1; j <= n; j++) {
            f[j][i] = f[f[j][i - 1]][i - 1];
            w[j][i] = min(w[j][i - 1], w[f[j][i - 1]][i - 1]);
        }
    }
    q = read();
    for(register int i = 1; i <= q; i++) {
        int x = read(), y = read();
        write(lca(x, y));
    }
    return 0;
}
```

