---
title: Luogu P2486 [SDOI2011]染色 解题报告
categories: 解题报告
date: 2019-07-19 09:37:17
tags:
	- 树上问题
	- 树链剖分
	- 线段树
	- 分类讨论
top:
---

傻逼题

# 题目链接

[**P2486** [SDOI2011]染色](https://www.luogu.org/problemnew/show/P2486) 

# 题意简述

直接搬luogu了。

![img](https://cdn.luogu.org/upload/pic/1600.png)

<!--more-->

# 解题思想

先树剖，线段树维护三个信息：

- 区间颜色数量
- 区间左端点颜色
- 区间右端点颜色

然后在合并的时候判一下两区间颜色是否重叠就行了。

做法一眼了，码了一万年。

难码，草。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <vector>
#include <algorithm>
#define int unsigned int

inline int read() {
    char ch = getchar(); int r = 0, w = 1;
    while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
    while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
    return r * w;
}

const int N = 100005;

int n, m, w[N], ut, vt, at, bt, ct;
char op;
int fa[N], siz[N], son[N], dfn[N], ptn[N], top[N], dep[N], cnt;
std::vector<int> gra[N];

inline void add(int u, int v) {
    gra[u].push_back(v);
    gra[v].push_back(u);
}

void dfs1(int u) {
    for(int i = 0; i < gra[u].size(); i++) {
        int v = gra[u][i];
        if(v == fa[u]) continue;
        dep[v] = dep[u] + 1;
        fa[v] = u;
        dfs1(v);
        siz[u] += siz[v] + 1;
        if(siz[v] > siz[son[u]]) son[u] = v;
    }
}

void dfs2(int u, int tp) {
    top[u] = tp;
    dfn[u] = ++cnt;
    ptn[dfn[u]] = u;
    if(son[u]) dfs2(son[u], tp);
    for(int i = 0; i < gra[u].size(); i++) {
        int v = gra[u][i];
        if(v == son[u] || v == fa[u]) continue;
        dfs2(v, v);
    }
}

struct Node {
    int l, r, tot;
    bool tag;
} sgt[N << 2];

void pushdown(int o) {
    int lch = o << 1, rch = (o << 1) | 1;
    if(sgt[o].tag) {
        sgt[lch].l = sgt[lch].r = sgt[rch].l = sgt[rch].r = sgt[o].l;
        sgt[lch].tag = sgt[rch].tag = true;
        sgt[lch].tot = sgt[rch].tot = 1;
        sgt[o].tag = false;
    }
}

void merge(Node &rt, Node ls, Node rs) {
    rt.l = ls.l;
    rt.r = rs.r;
    rt.tot = ls.tot + rs.tot;
    if(ls.r == rs.l) rt.tot--;
}

void build(int o, int l, int r) {
    if(l == r) {
        sgt[o].l = sgt[o].r = w[ptn[l]];
        sgt[o].tot = 1;
        return;
    }
    int mid = (l + r) >> 1, lch = o << 1, rch = (o << 1) | 1;
    build(lch, l, mid);
    build(rch, mid + 1, r);
    merge(sgt[o], sgt[lch], sgt[rch]);
}

void modify(int o, int l, int r, int ll, int rr, int v) {
    if(l == ll && r == rr) {
        sgt[o].l = sgt[o].r = v;
        sgt[o].tot = 1;
        sgt[o].tag = true;
        return;
    }
    pushdown(o);
    int mid = (l + r) >> 1, lch = o << 1, rch = (o << 1) | 1;
    if(rr <= mid) {
        modify(lch, l, mid, ll, rr, v);
    } else if(ll > mid) {
        modify(rch, mid + 1, r, ll, rr, v);
    } else {
        modify(lch, l, mid, ll, mid, v);
        modify(rch, mid + 1, r, mid + 1, rr, v);
    }
    merge(sgt[o], sgt[lch], sgt[rch]);
}

Node query(int o, int l, int r, int ll, int rr) {
    if(l == ll && r == rr) {
        return sgt[o];
    }
    pushdown(o);
    int mid = (l + r) >> 1, lch = o << 1, rch = (o << 1) | 1;
    if(rr <= mid) {
        return query(lch, l, mid, ll, rr);
    } else if(ll > mid) {
        return query(rch, mid + 1, r, ll, rr);
    } else {
        Node ls = query(lch, l, mid, ll, mid), rs = query(rch, mid + 1, r, mid + 1, rr), res;
        merge(res, ls, rs);
        return res;
    }
}

void modify(int u, int v, int c) {
    int tu = top[u], tv = top[v];
    while(tu != tv) {
        if(dep[tu] > dep[tv]) {
            std::swap(tu, tv);
            std::swap(u, v);
        }
        modify(1, 1, n, dfn[tv], dfn[v], c);
        v = fa[tv];
        tv = top[v];
    }
    if(dep[u] > dep[v]) std::swap(u, v);
    modify(1, 1, n, dfn[u], dfn[v], c);
}

int query(int u, int v) {
    Node t;
    int res = 0, cu = -1, cv = -1, tu = top[u], tv = top[v];
    while(tu != tv) {
        if(dep[tu] > dep[tv]) {
            std::swap(tu, tv);
            std::swap(u, v);
            std::swap(cu, cv);
        }
        t = query(1, 1, n, dfn[tv], dfn[v]);
        res += t.tot;
        if(t.r == cv) res--;
        cv = t.l;
        v = fa[tv];
        tv = top[v];
    }
    if(dep[u] > dep[v]) {
        std::swap(u, v);
        std::swap(cu, cv); 
    }
    t = query(1, 1, n, dfn[u], dfn[v]);
    res += t.tot;
    if(t.l == cu) res--;
    if(t.r == cv) res--;
    return res;
}

signed main() {
    n = read(); m = read();
    for(int i = 1; i <= n; i++) w[i] = read();
    for(int i = 1; i < n; i++) {
        ut = read(); vt = read();
        add(ut, vt);
    }
    dfs1(1);
    dfs2(1, 1);
    build(1, 1, n);
    while(m--) {
        do {
            op = getchar();
        } while(op != 'C' && op != 'Q');
        at = read();
        bt = read();
        if(op == 'Q') {
            printf("%d\n", query(at, bt));
        }
        if(op == 'C') {
            ct = read();
            modify(at, bt, ct);
        }
    }
    return 0;
}
```

