---
title: Luogu P2617 Dynamic Rankings 解题报告
categories: 解题报告
toc: true
date: 2019-09-20 18:47:32
tags:
	- 可持久化数据结构
	- 主席树
	- 树状数组
	- 树套树
top:
thumbnail: https://i.loli.net/2019/10/08/19BbYg3CMQsFd2o.jpg
---

# 题目链接

[P2617 Dynamic Rankings](https://www.luogu.org/problem/P2617)

# 题意简述

动态区间第 $k$ 小。

<!--more-->

# 解题思想

考虑静态解法，单点修改后对对应的主席树暴力，单次修改复杂度 $O(n\log n)$。显然会T。

考虑主席树维护的前缀和，我们发现问题相当于单点修改，区间查询。

这个不难用树状数组维护。

所以可以树状数组套主席树解决，单次修改 $O(\log^2n)$。

需要注意的是，必须把预处理所有的待修改元素，所以这是个离线算法。

修改可以将树状数组的 `modify()` 中的 `+=` 直接改成主席树 `modify()`，但是查询需要 $O(\log n)$ 个节点一起向下移动。所以要在主席树 `modify()` 的内部处理。

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

const int N = 4e7 + 6;

int n, m, sz, idx;
int a[N], b[N], t[N], ls[N], rs[N], rt[N], cur1[N], cur2[N], q1[N], q2[N], q3[N];
bool type[N];
#define mid ((l + r) >> 1)
inline void copy(int old, int o) {t[o] = t[old], ls[o] = ls[old], rs[o] = rs[old];}
inline void upd(int o) {t[o] = t[ls[o]] + t[rs[o]];}
void build(int &o, int l, int r) {o = ++idx; if(l == r) return; build(ls[o], l, mid); build(rs[o], mid + 1, r);}
void modify(int old, int &o, int l, int r, int x, int k) {o = ++idx; copy(old, o); if(l == r) {t[o] += k; return;} if(x <= mid) modify(ls[old], ls[o], l, mid, x, k); else modify(rs[old], rs[o], mid + 1, r, x, k); upd(o);}
inline int lowbit(int x) {return x & -x;}
inline void modify(int o, int x, int k) {for(; o <= n; o += lowbit(o)) modify(rt[o], rt[o], 1, sz, x, k);}
inline int query(int ll, int rr, int k) {
    int l = 1, r = sz, sum1, sum2;
    for(register int i = ll; i; i -= lowbit(i)) cur1[i] = rt[i];
    for(register int i = rr; i; i -= lowbit(i)) cur2[i] = rt[i];
    while(l < r) {
        sum1 = sum2 = 0;
        for(register int i = ll; i; i -= lowbit(i)) sum1 += t[ls[cur1[i]]];
        for(register int i = rr; i; i -= lowbit(i)) sum2 += t[ls[cur2[i]]];
        if(sum2 - sum1 >= k) {
            for(register int i = ll; i; i -= lowbit(i)) cur1[i] = ls[cur1[i]];
            for(register int i = rr; i; i -= lowbit(i)) cur2[i] = ls[cur2[i]];
            r = mid;
        }
        else {
            for(register int i = ll; i; i -= lowbit(i)) cur1[i] = rs[cur1[i]];
            for(register int i = rr; i; i -= lowbit(i)) cur2[i] = rs[cur2[i]];
            l = mid + 1, k -= sum2 - sum1;
        }
    }
    return l;
}

bool isQ() {
    char op = getchar();
    while(op != 'Q' && op != 'C') op = getchar();
    return op == 'Q';
}

int main() {
    n = read(), m = read(), sz = n;
    for(register int i = 1; i <= n; i++) a[i] = b[i] = read();
    for(register int i = 1; i <= m; i++) {
        type[i] = isQ();
        q1[i] = read(), q2[i] = read();
        if(type[i]) q3[i] = read();
        else b[++sz] = q2[i];
    }
    std::sort(b + 1, b + sz + 1);
    sz = std::unique(b + 1, b + sz + 1) - b - 1;
    build(rt[0], 1, sz);
    for(register int i = 1; i <= n; i++) modify(i, std::lower_bound(b + 1, b + sz + 1, a[i]) - b, 1);
    for(register int i = 1; i <= m; i++) {
        if(type[i]) printf("%d\n", b[query(q1[i] - 1, q2[i], q3[i])]);
        else {
            modify(q1[i], std::lower_bound(b + 1, b + sz + 1, a[q1[i]]) - b, -1);
            a[q1[i]] = q2[i];
            modify(q1[i], std::lower_bound(b + 1, b + sz + 1, a[q1[i]]) - b, 1);
        }
    }
    return 0;
}
```
