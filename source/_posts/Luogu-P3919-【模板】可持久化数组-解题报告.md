---
title: Luogu P3919 【模板】可持久化数组 解题报告
categories: 解题报告
toc: true
date: 2019-09-19 20:11:10
tags:
	- 可持久化数据结构
top:
thumbnail: https://i.loli.net/2019/09/19/OmGlsj9SQd3H8Dc.jpg
---

# 题目链接

[P3919 【模板】可持久化数组（可持久化线段树/平衡树）](https://www.luogu.org/problem/P3919)

# 题意简述

给你一个序列，进行如下操作：

-   修改某位置元素
-   查询第 `i` 次操作后的某位置元素

<!--more-->

# 解题思想

可持久化线段树板子题。

就是在查询的时候也新建版本就行了。

注意每次 `modify` 和 `query` 的是 `rt[old]` 而不是 `old`。

数组要开大...以后无脑 $50$ 倍吧...

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

const int N = 2e6 + 6;

int n, m, tot;
int a[N], ls[N * 20], rs[N * 20], rt[N * 20], t[N * 20], idx;
#define mid ((l + r) >> 1)

void build(int &o, int l, int r) {
	o = ++idx;
	if(l == r) {t[o] = a[l]; return;}
	build(ls[o], l, mid);
	build(rs[o], mid + 1, r);
}

void modify(int old, int &o, int l, int r, int x, int k) {
	o = ++idx;
	ls[o] = ls[old], rs[o] = rs[old], t[o] = t[old];
	if(l == r) {t[o] = k; return;}
	if(x <= mid) modify(ls[old], ls[o], l, mid, x, k);
	else modify(rs[old], rs[o], mid + 1, r, x, k);
}

int query(int old, int &o, int l, int r, int x) {
	o = ++idx;
	ls[o] = ls[old], rs[o] = rs[old], t[o] = t[old];
	if(l == r) return t[o];
	if(x <= mid) return query(ls[old], ls[o], l, mid, x);
	else return query(rs[old], rs[o], mid + 1, r, x);
}

int main() {
	n = read(), m = read();
	for(register int i = 1; i <= n; i++) a[i] = read();
	build(rt[tot++], 1, n);
	while(m--) {
		int old = read(), op = read();
		if(op == 1) {
			int x = read(), k = read();
			modify(rt[old], rt[tot++], 1, n, x, k);
		}
		else {
			printf("%d\n", query(rt[old], rt[tot++], 1, n, read()));
		}
	}
	return 0;
}
```

