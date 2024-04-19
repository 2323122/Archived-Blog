---
title: Luogu P2634 [国家集训队]聪聪可可 解题报告
categories: 解题报告
toc: true
date: 2019-09-23 19:33:41
tags:
	- 树上问题
	- 点分治
top:
thumbnail: https://i.loli.net/2019/09/23/9qLBu2JEpjFIwfZ.jpg
---

算是入门淀粉质了吧。

# 题目链接

[P2634 [国家集训队]聪聪可可](https://www.luogu.org/problem/P2634)

# 题意简述

问树上简单路径长度为 $3$ 的倍数的概率。

<!--more-->

# 解题思想

先把路径长度对 $3$ 取模，在点分治的过程中算出所有到当前根距离 "模 $3$ 余 $0$"，"模 $3$ 余 $1$"，"模 $3$ 余 $2$" 的简单路径数，分别记为 $dis_1, dis_2, dis_3$ ，那么当前根的贡献就是 $dis_1^2 + 2\times dis_2\times dis_3$。剩下的按点分正常做。

总的方案数为 $n^2$（随便选两个点），最后约分输出就行了。

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

template <typename T> inline T min(T a, T b) {return a < b ? a : b;}
template <typename T> inline T max(T a, T b) {return a > b ? a : b;}
template <typename T> inline void chkmin(T &a, T b) {a = min(a, b);}
template <typename T> inline void chkmax(T &a, T b) {a = max(a, b);}

const int N = 20010;
const int M = N << 1;

int n, m, rt, S, ans;
int head[N], ver[M], nxt[M], edge[M], cnt;
int a[N], dis[3];
int sz[N], mx[N];
bool v[N];

int gcd(int x, int y) {return y ? gcd(y, x % y) : x;}

inline void add(int x, int y, int z) {
	ver[++cnt] = y, edge[cnt] = z, nxt[cnt] = head[x], head[x] = cnt;
}

void find(int x, int fa) {
	sz[x] = 1, mx[x] = 0;
	for(register int i = head[x]; i; i = nxt[i]) {
		int y = ver[i];
		if(v[y] || y == fa) continue;
		find(y, x);
		sz[x] += sz[y];
		chkmax(mx[x], sz[y]);
	}
	chkmax(mx[x], S - sz[x]);
	if(mx[x] < mx[rt]) rt = x;
}

void dfs(int x, int fa) {
	dis[a[x] % 3]++;
	for(register int i = head[x]; i; i = nxt[i]) {
		int y = ver[i], z = edge[i];
		if(y == fa || v[y]) continue;
		a[y] = a[x] + z;
		dfs(y, x);
	}
}

int solve(int s, int len) {
	a[s] = len;
	dis[0] = dis[1] = dis[2] = 0;
	dfs(s, 0);
	return dis[0] * dis[0] + dis[1] * dis[2] * 2;
}

void divide(int x) {
	ans += solve(x, 0);
	v[x] = 1;
	for(register int i = head[x]; i; i = nxt[i]) {
		int y = ver[i], z = edge[i];
		if(v[y]) continue;
		ans -= solve(y, z);
		S = sz[y], rt = 0;
		find(y, x);
		divide(rt);
	}
}

int main() {
	n = read();
	for(register int i = 1; i < n; i++) {
		int x = read(), y = read(), z = read() % 3;
		add(x, y, z);
		add(y, x, z);
	}
	S = n, rt = 0, mx[0] = n;
	find(1, 0);
	divide(rt);
	int up = gcd(ans, n * n);
	printf("%d/%d\n", ans / up, n * n / up);
	return 0;
}
```

