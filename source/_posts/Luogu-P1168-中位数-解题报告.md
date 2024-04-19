---
title: Luogu P1168 中位数 解题报告
categories: 解题报告
toc: true
date: 2019-09-20 11:01:30
tags:
	- 可持久化数据结构
top:
thumbnail: https://i.loli.net/2019/09/20/AfMwv49UslOhELi.jpg
---

水了道绿题，美滋滋。

# 题目链接

[P1168 中位数](https://www.luogu.org/problem/P1168)

# 题意简述

给你一个序列，求前奇数项的中位数。

<!--more-->

# 解题思想

前 $k$ 项的中位数（$k$ 为奇数）== 前 $k$ 个数中第 $(k + 1) /2$ 大的值。

静态区间第 $k$ 大，可以用主席树写。

序列中的数字不是连续的，所以需要离散化一下。

做完了。

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

int n, sz, a[N], b[N];
int t[N << 5], ls[N << 5], rs[N << 5], rt[N << 5], idx;
#define mid ((l + r) >> 1)
void copy(int old, int o) {t[o] = t[old], ls[o] = ls[old], rs[o] = rs[old];}
void upd(int o) {t[o] = t[ls[o]] + t[rs[o]];}
void build(int &o, int l, int r) {o = ++idx; if(l == r) return; build(ls[o], l, mid); build(rs[o], mid + 1, r);}
void modify(int old, int &o, int l, int r, int k) {o = ++idx; copy(old, o); if(l == r) {t[o]++; return;} if(k <= mid) modify(ls[old], ls[o], l, mid, k); else modify(rs[old], rs[o], mid + 1, r, k); upd(o);}
int query(int o, int l, int r, int k) {if(l == r) return l; if(k <= t[ls[o]]) return query(ls[o], l, mid, k); else return query(rs[o], mid + 1, r, k - t[ls[o]]);}

int main() {
    n = read();
    for(register int i = 1; i <= n; i++) a[i] = b[i] = read();
    std::sort(b + 1, b + n + 1);
    sz = std::unique(b + 1, b + n + 1) - b - 1;
    build(rt[0], 1, sz);
    for(register int i = 1; i <= n; i++) modify(rt[i - 1], rt[i], 1, sz, std::lower_bound(b + 1, b + sz + 1, a[i]) - b);
    for(register int i = 1; i <= n; i += 2) printf("%d\n", b[query(rt[i], 1, sz, (i + 1) / 2)]);
    return 0;
}
```

