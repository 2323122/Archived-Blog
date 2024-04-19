---
title: Luogu P4556 [Vani有约会]雨天的尾巴 解题报告
categories: 解题报告
toc: true
date: 2019-10-15 18:51:03
tags:
	- 图论
	- 树上问题
	- 数据结构
	- 线段树
	- 线段树合并
	- 离散化
	- 树上差分
	- LCA
top:
thumbnail: https://i.loli.net/2019/10/15/cFENtZVgKzDmr3I.jpg
---

人生成就题×
码农题√

# 题目链接

 [P4556 [Vani有约会]雨天的尾巴](https://www.luogu.org/problem/P4556) 

# 题意简述

给你一棵树，树上有 $n$ 个节点，$m$ 次操作，每次给一条链染色，询问每个节点被覆盖最多次数的颜色。

<!--more-->

# 解题思想

首先通过**链加**，**单点查**，不难看出要**树上差分**。然后考虑高效地处理**每个节点的颜色**信息：

覆盖最多次数的颜色显然可以通过**线段树**来维护：
我们对于树上的一个节点，开一颗**权值线段树**，点的**位置**（即权值）表示点的**颜色**。对于每个区间，我们维护一下**当前区间覆盖最多次数的颜色**以及**该颜色被覆盖的次数**。线段树 $pushup$ 就可以写成这样：

```c++
// ans[o] 表示节点 o 表示的区间中，覆盖次数最多的颜色。
// t[o] 表示该颜色被覆盖的次数。
//向上合并时将左右子区间答案比较并更新即可。
void upd(int o) {
    t[o] = max(t[ls[o]], t[rs[o]]);
    ans[o] = t[ls[o]] >= t[rs[o]] ? ans[ls[o]] : ans[rs[o]];
}
```

注：为了后面的操作，线段树使用动态开点。

**差分时**，对于树上的每一个点，我们**无需单独建线段树**（这样的话是否动态开点就没有区别了），而是**直接修改**对应根对应颜色的信息，然后向上更新。

而**差分后**，使用 `dfs` 对答案进行统计时，由儿子向父节点信息的合并也由加法变成了**线段树合并**。将儿子节点的覆盖次数添加到父节点，然后正常更新父节点即可。

每个节点的答案即为**当前节点对应的线段树的根节点**维护的 `ans` 信息。

需要注意的是，由于题目中**颜色数**远大于**操作数**，所以我们需要先将颜色**离散化**。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <algorithm>

inline int read() {
    char ch = getchar(); int r = 0, w = 1;
    while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
    while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
    return r * w;
}

const int N = 1e5 + 6;
const int M = N << 1;

inline int max(int a, int b) {return a >= b ? a : b;}

int n, m;
int head[N], ver[M], nxt[M], cnt;
int d[N], fa[N], sz[N], tp[N], son[N], f[N];

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
    if(son[x]) dfs2(son[x], top);
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(y == fa[x] || y == son[x]) continue;
        dfs2(y, y);
    }
}

int lca(int x, int y) {
    while(tp[x] != tp[y]) {
        if(d[tp[x]] > d[tp[y]]) std::swap(x, y);
        y = fa[tp[y]];
    }
    return d[x] < d[y] ? x : y;
}

struct Node {
    int x, y, z, k;
    Node() {}
    Node(int _x, int _y, int _k) : x(_x), y(_y), z(lca(_x, _y)), k(_k) {}
} q[N];

int a[N];
int rt[N << 6], t[N << 6], ans[N << 6], ls[N << 6], rs[N << 6], tot;

#define mid ((l + r) >> 1)

void upd(int o) {t[o] = max(t[ls[o]], t[rs[o]]), ans[o] = t[ls[o]] >= t[rs[o]] ? ans[ls[o]] : ans[rs[o]];}

void modify(int &o, int l, int r, int x, int k) {
    if(!o) o = ++tot;
    if(l == r) return t[o] += k, ans[o] = t[o] ? l : 0, void();
    if(x <= mid) modify(ls[o], l, mid, x, k);
    else modify(rs[o], mid + 1, r, x, k);
    upd(o);
}

int merge(int &p, int q, int l, int r) {
    if(!p || !q) return p ^ q;
    if(l == r) {
        t[p] += t[q];
        ans[p] = t[p] ? l : 0;
        return p;
    }
    ls[p] = merge(ls[p], ls[q], l, mid);
    rs[p] = merge(rs[p], rs[q], mid + 1, r);
    upd(p);
    return p;
}

void dfs(int x) {
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(fa[x] == y) continue;
        dfs(y);
        rt[x] = merge(rt[x], rt[y], 1, cnt);
    }
    f[x] = ans[rt[x]];
}

int main() {
    n = read(), m = read();
    for(register int i = 1; i < n; i++) {
        int x = read(), y = read();
        add(x, y);
    }
    dfs1(1);
    dfs2(1, 1);
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read(), k = read();
        q[i] = Node(x, y, k);
    }
    for(register int i = 1; i <= m; i++) a[i] = q[i].k;
    std::sort(a + 1, a + m + 1);
    cnt = std::unique(a + 1, a + m + 1) - a - 1;
    for(register int i = 1; i <= m; i++) {
        int x = q[i].x, y = q[i].y, z = q[i].z, k = std::lower_bound(a + 1, a + cnt + 1, q[i].k) - a;
        modify(rt[x], 1, cnt, k, 1);
        modify(rt[y], 1, cnt, k, 1);
        modify(rt[z], 1, cnt, k, -1);
        if(fa[z]) modify(rt[fa[z]], 1, cnt, k, -1);
    }
    dfs(1);
    for(register int i = 1; i <= n; i++) {
        printf("%d\n", a[f[i]]);
    }
    return 0;
}
```

