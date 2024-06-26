---
title: Luogu P2146 [NOI2015]软件包管理器 解题报告
categories: 解题报告
date: 2019-07-17 19:07:38
tags: 树上问题
top:
---



![](https://i.loli.net/2019/07/17/5d2f025c5513d85813.png)

这是[wjyyy](https://www.wjyyy.top/3450.html#Day_1)嗦的$\uparrow$

# 题目链接

[**P2146** [NOI2015]软件包管理器](https://www.luogu.org/problemnew/show/P2146) 

# 题意简述

给你一棵树，有点权。将点权赋值为1需要将根到该点路径上所有点赋值为，将点权赋值为0会使以该点为根的子树上的所有点点权变为0。初始时所有点点权为0，给你m个赋值操作，输出每个操作会改变多少个点的状态。

<!--more-->

# 解题思想

发现树的形态没有改变，我们考虑树剖。然后求出根到某个点的链的长度和某个点的子树大小，线段树维护即可。

但是我们发现这里面是区间赋值操作，而不是区间增加操作，如果单个点做效率极低，我们考虑一种新的线段树：

## 区间覆盖线段树

我们考虑在区间增线段树的基础上将**增加**改成**赋值**。

于是我们将`tag[]`维护的信息由增量改为赋值数，发现根据时间的先后，这个信息会被逐步**替代**，于是就可以在下放的时候**直接用父节点标记盖住子节点标记**，即把`+=`改为`=`。这样就可以**整个区间**赋值为**同一个数**。

注意一下题目中有赋值为`0`的操作，所以`tag[]`的初值**不能**为0，否则就会出锅。（卡了我一下午

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <cstring>

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

const int N = 100010;
const int M = N << 1;

template <typename T> inline void swap(T &x, T &y) {T tmp = x; x = y, y = tmp;}
template <typename T> inline T min(T a, T b) {return a < b ? a : b;}
template <typename T> inline T max(T a, T b) {return a > b ? a : b;}

int n, q, w[N];
int head[N], nxt[M], ver[M], cnt, idx;
int d[N], sz[N], fa[N], tp[N], son[N];
int dfn[N], ptn[N];
int val[N << 2], tag[N << 2];

#define ls (o << 1)
#define rs (o << 1 | 1)
#define mid ((l + r) >> 1)

inline void upd(int o) {
    val[o] = val[ls] + val[rs];
}

inline void pushdown(int o, int l, int r) {
    if(tag[o] != -1) {
        tag[ls] = tag[o];
        tag[rs] = tag[o];
        val[ls] = tag[o] * (mid - l + 1);
        val[rs] = tag[o] * (r - mid);
    }
    tag[o] = -1;
}

void build(int o, int l, int r) {
    if(l == r) {val[o] = w[ptn[l]]; return;}
    build(ls, l, mid);
    build(rs, mid + 1, r);
    upd(o);
}

void modify(int o, int l, int r, int ll, int rr, int k) {
    if(l >= ll && r <= rr) {
        val[o] = k * (r - l + 1);
        tag[o] = k;
        return;
    }
    pushdown(o, l, r);
    if(ll <= mid) modify(ls, l, mid, ll, rr, k);
    if(rr > mid) modify(rs, mid + 1, r, ll, rr, k);
    upd(o);
}

int query(int o, int l, int r, int ll, int rr) {
    if(l >= ll && r <= rr) return val[o];
    pushdown(o, l, r);
    int res = 0;
    if(ll <= mid) res += query(ls, l, mid, ll, rr);
    if(rr > mid) res += query(rs, mid + 1, r, ll, rr);
    return res;
}

void add(int x, int y) {
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
    ptn[idx] = x;
    if(son[x]) dfs2(son[x], top);
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(y == fa[x] || y == son[x]) continue;
        dfs2(y, y);
    }
}

inline void changeline(int x, int y, int z) {
    while(tp[x] != tp[y]) {
        if(d[tp[x]] > d[tp[y]]) swap(x, y);
        modify(1, 1, n, dfn[tp[y]], dfn[y], z);
        y = fa[tp[y]];
    }
    if(d[x] > d[y]) swap(x, y);
    modify(1, 1, n, dfn[x], dfn[y], z);
}

inline int queryline(int x, int y) {
    int res = 0;
    while(tp[x] != tp[y]) {
        if(d[tp[x]] > d[tp[y]]) swap(x, y);
        res += query(1, 1, n, dfn[tp[y]], dfn[y]);
        y = fa[tp[y]];
    }
    if(d[x] > d[y]) swap(x, y);
    res += query(1, 1, n, dfn[x], dfn[y]);
    return res;
}

inline void changetree(int x, int z) {
    modify(1, 1, n, dfn[x], dfn[x] + sz[x] - 1, z);
}

inline int querytree(int x) {
    return query(1, 1, n, dfn[x], dfn[x] + sz[x] - 1);
}

char s[10];

int main() {
    memset(tag, -1, sizeof(tag));
    n = read();
    for(register int i = 2; i <= n; i++) {
        add(read() + 1, i);
    }
    dfs1(1);
    dfs2(1, 1);
    build(1, 1, n);
    q = read();
    for(register int i = 1; i <= q; i++) {
        scanf("%s", s);
        register int x = read() + 1;
        int num = val[1];
        if(s[0] == 'i') {
            changeline(1, x, 1);
            write(val[1] - num);
        }
        else {
            changetree(x, 0);
            write(num - val[1]);
        }
    }
    return 0;
}
```

