---
title: Luogu P3258 [JLOI2014]松鼠的新家 解题报告
categories: 解题报告
date: 2019-07-18 09:12:06
tags:
	- 树上问题
	- 树上差分
	- 树链剖分
top:
---

线段树大常数选手怕被卡，所以写了树上差分qwq。

# 题目链接

[**P3258** [JLOI2014]松鼠的新家](https://www.luogu.org/problemnew/show/P3258) 

# 题意简述

给你一棵树和一个遍历顺序，求每个点至少经过多少次。

<!--more-->

# 解题思想

## 树剖+线段树

直接对每条链区间加1就可，板子题，两个log~~跑得快~~。

## 树剖+树上差分

考虑只有链，还是加1，不妨用树上差分~~然而写这题之前我并不会~~，对于每条链，我们考虑维护一个到根距离的差分数组记为`d[x]`，这样的话，要对链$(x,y)$整体加一，只需`d[x]++, d[y]++, d[fa[lca(x, y)]] -= 2`，把祖先节点到根的距离容斥掉，即可完成对链的整体加1，少个log。

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

const int N = 3e5 + 6;
const int M = N << 1;

template <typename T> inline void swap(T &x, T &y) {T tmp = x; x = y, y = tmp;}

int n, delta[N];
int head[N], nxt[M], ver[M], cnt;
int w[N], a[N], dfn[N], ptn[N], idx;
int tp[N], sz[N], d[N], fa[N], son[N];

inline void add(int x, int y) {
    ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
    ver[++cnt] = x, nxt[cnt] = head[y], head[y] = cnt;
}

void dfs1(int x) {
    sz[x] = 1;
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(y == fa[x]) continue;
        fa[y] = x;
        d[y] = d[x] + 1;
        dfs1(y);
        sz[x] += sz[y];
        if(sz[y] > sz[son[x]]) son[x] = y;
    }
}

void dfs2(int x, int top) {
    tp[x] = top;
    dfn[x] = ++idx;
    ptn[dfn[x]] = x;
    if(son[x]) dfs2(son[x], top);
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(y == fa[x] || y == son[x]) continue;
        dfs2(y, y);
    }
}

inline int lca(int x, int y) {
    while(tp[x] != tp[y]) {
        if(d[tp[x]] > d[tp[y]]) swap(x, y);
        y = fa[tp[y]];
    }
    return d[x] < d[y] ? x : y;
}

int main() {
    n = read();
    for(register int i = 1; i <= n; i++) w[i] = read();
    for(register int i = 1; i < n; i++) add(read(), read());
    dfs1(1);
    dfs2(1, 1);
    a[w[1]] = 1;
    for(register int i = 2; i <= n; i++) {
        delta[w[i - 1]]++;
        delta[w[i]]++;
        delta[fa[lca(w[i - 1], w[i])]] -= 2;
        a[lca(w[i - 1], w[i])]--;
    }
    for(register int i = n; i >= 1; i--)
        a[ptn[i]] += delta[ptn[i]], delta[fa[ptn[i]]] += delta[ptn[i]];
    for(register int i = 1; i <= n; i++)
        write(a[i] - 1);
    return 0;
}
```

