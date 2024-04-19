---
title: Luogu P2824 [HEOI2016/TJOI2016]排序 解题报告
categories: 解题报告
toc: true
date: 2019-10-16 09:53:42
tags:
	- 线段树
	- 二分答案
top:
thumbnail: https://i.loli.net/2019/10/16/JXa12LDPR4SpQYt.png
---

让人眼前一亮的新科技get√

# 题目链接

[P2824 [HEOI2016/TJOI2016]排序](https://www.luogu.org/problem/P2824)

# 题意简述

给你一个序列，区间排序 $m$ 次后询问第 $p$ 个位置上的数。

<!--more-->

# 解题思想

直接排序是 $O(n\log n)$ 的，模拟显然会 T。我们考虑降低排序的复杂度。

注意到最后只查询一个位置的元素，我们不妨二分这个值，然后将原序列转为储存元素与该值大小关系的 $01$ 序列。

然后考虑 $01$ 序列排序，这个是可以 $O(\log n)$ 实现的。只需要对序列建线段树，然后统计区间 $1$ 的个数，并按序区间赋值即可。

查询的时候直接线段树单点查询 $p$ 位置的元素是否为 $1$ 以调整二分上下界。

# 参考代码

```c++
#include <cstdio>
#include <cctype>

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

const int N = 1e5 + 6;

int n, m, pos, ans;
int a[N];
int t[N << 2], tag[N << 2];

#define ls (o << 1)
#define rs (o << 1 | 1)
#define mid ((l + r) >> 1)

void upd(int o) {t[o] = t[ls] + t[rs];}

void build(int o, int l, int r, int x) {
	if(l == r) return t[o] = a[l] >= x, tag[o] = -1, void();
	build(ls, l, mid, x);
	build(rs, mid + 1, r, x);
	upd(o), tag[o] = -1;
}

void pushdown(int o, int l, int r) {
	if(~tag[o]) {
		tag[ls] = tag[rs] = tag[o];
		if(tag[o]) t[ls] = mid - l + 1, t[rs] = r - mid;
		else t[ls] = t[rs] = 0;
		tag[o] = -1;
	}
}

int query(int o, int l, int r, int ll, int rr) {
	if(ll > rr) return 0;
	if(ll <= l && rr >= r) return t[o];
	pushdown(o, l, r);
	int res = 0;
	if(ll <= mid) res += query(ls, l, mid, ll, rr);
	if(rr > mid) res += query(rs, mid + 1, r, ll, rr);
	return res;
}

int query(int o, int l, int r, int x) {
	if(l == r) return t[o];
	pushdown(o, l, r);
	if(x <= mid) return query(ls, l, mid, x);
	else return query(rs, mid + 1, r, x);
}

void modify(int o, int l, int r, int ll, int rr, int k) {
	if(ll > rr) return;
	if(ll <= l && rr >= r) return t[o] = k * (r - l + 1), tag[o] = k, void();
	pushdown(o, l, r);
	if(ll <= mid) modify(ls, l, mid, ll, rr, k);
	if(rr > mid) modify(rs, mid + 1, r, ll, rr, k);
	upd(o);
}

struct opt {
	int l, r, op;
	opt() {}
	opt(int _l, int _r, int _op) : l(_l), r(_r), op(_op) {}
} q[N];

bool check(int x) {
	build(1, 1, n, x);
	for(register int i = 1; i <= m; i++) {
		int len = query(1, 1, n, q[i].l, q[i].r);
		if(!q[i].op) {
			modify(1, 1, n, q[i].l, q[i].r - len, 0);
			modify(1, 1, n, q[i].r - len + 1, q[i].r, 1);
		}
		else {
			modify(1, 1, n, q[i].l, q[i].l + len - 1, 1);
			modify(1, 1, n, q[i].l + len, q[i].r, 0);
		}
	}
	return query(1, 1, n, pos);
}

int main() {
	n = read(), m = read();
	for(register int i = 1; i <= n; i++) a[i] = read();
	for(register int i = 1; i <= m; i++) {
		int op = read(), l = read(), r = read();
		q[i] = opt(l, r, op);
	}
	pos = read();
	int l = 1, r = n;
	while(l < r) {
		if(check(mid)) ans = mid, l = mid + 1;
		else r = mid;
	}
	printf("%d\n", ans);
	return 0;
}
```

