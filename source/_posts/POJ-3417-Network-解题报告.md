---
title: POJ 3417 Network 解题报告
categories: 解题报告
toc: true
date: 2019-10-15 15:30:50
tags:
	- 图论
	- 计数
	- 树上差分
	- 分类讨论
top:
thumbnail: https://i.loli.net/2019/10/15/yFC64qJfOdTom2N.jpg
---

树上差分恶补计划 on

# 题目链接

[Network]( http://poj.org/problem?id=3417 )

# 题意简述

给你一颗树和 $m$ 条附加边，要求删去一条树边，删去一条附加边，使原树不连通。求方案数。

<!--more-->

# 解题思想

注意到添加附加边相当于在两端点的简单路径上再覆盖一遍，不妨对每条树边被附加边覆盖的次数分类讨论：

-   若一条树边未被覆盖过，则将其删去可导致原树直接不连通，贡献了 $m$ 种方案。
-   若一条树边被覆盖了一次，则将原树边和覆盖它的这条附加边同时删去可导致原树不连通，贡献了 $1$ 种方案。
-   若一条树边被覆盖了超过一次，则无论如何都无法使原树不连通，无贡献。

树上差分后按边统计即可。

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

int n, m, ans;
int head[N], ver[M], nxt[M], cnt;
int f[N], d[N], fa[N], sz[N], tp[N], son[N];

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

void dfs(int x) {
	for(register int i = head[x]; i; i = nxt[i])
		if(fa[x] == ver[i]) continue;
		else dfs(ver[i]), f[x] += f[ver[i]];
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
		int x = read(), y = read(), z = lca(x, y);
		f[x]++, f[y]++, f[z] -= 2;
	}
	dfs(1);
	for(register int i = 2; i <= n; i++) if(f[i] == 0) ans += m; else if(f[i] == 1) ans++;
	printf("%d\n", ans);
	return 0;
}
```

