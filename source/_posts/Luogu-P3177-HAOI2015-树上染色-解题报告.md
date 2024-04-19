---
title: Luogu P3177 [HAOI2015]树上染色 解题报告
categories: 解题报告
toc: true
date: 2019-10-14 16:49:43
tags:
	- DP
	- 树形DP
	- 树上背包
top:
thumbnail: https://i.loli.net/2019/10/14/zV7vrQWXZJIyO9P.png
---

~~懒得找图所以很久没写博客了..~~

# 题目链接

[P3177 [HAOI2015]树上染色](https://www.luogu.org/problem/P3177)

# 题意简述

给你一棵树，有边权，将 $k$ 个点染成黑色，剩下的染成白色，总贡献为黑点两两间的路径长度之和加上白点两两间的路径长度之和。求最大贡献。

<!--more-->

# 解题思想

考虑 $dp$ 某条边的贡献。对于一条边，我们可以发现，当一种染色方案确定时，它对最后答案的贡献为边两端黑点数的乘积乘以边权+边两端白点数的乘积乘以边权。

那么我们可以对每条边的贡献DP。

设 $dp[x][i]$ 为以 $x$ 为根的子树中，将 $i$ 个点染成黑色对答案的贡献。

那么直接树上背包向根节点转移即可。

注意开 long long。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <cstring>

#define int long long

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

const int N = 2e3 + 6;
const int M = N << 1;

int n, k;
int head[N], ver[M], nxt[M], edge[M], sz[N], cnt;
int dp[N][N];

inline int min(int a, int b) {return a < b ? a : b;}
inline int max(int a, int b) {return a > b ? a : b;}

inline void add(int x, int y, int z) {
	ver[++cnt] = y, edge[cnt] = z, nxt[cnt] = head[x], head[x] = cnt;
	ver[++cnt] = x, edge[cnt] = z, nxt[cnt] = head[y], head[y] = cnt;
}

void dfs(int x, int fa) {
	sz[x] = 1;
	memset(dp[x], -1, sizeof dp[x]);
	dp[x][0] = dp[x][1] = 0;
	for(register int i = head[x]; i; i = nxt[i]) {
		int y = ver[i];
		if(y == fa) continue;
		dfs(y, x);
		sz[x] += sz[y];
	}
	for(register int i = head[x]; i; i = nxt[i]) {
		int y = ver[i], z = edge[i];
		if(y == fa) continue;
		for(register int i = min(k, sz[x]); i >= 0; i--)
			for(register int j = 0; j <= min(i, sz[y]); j++)
				if(~dp[x][i - j]) dp[x][i] = max(dp[x][i], dp[x][i - j] + dp[y][j] + j * (k - j) * z + (sz[y] - j) * (n - k + j - sz[y]) * z);
	}
}

signed main() {
	n = read(), k = read();
	for(register int i = 1; i < n; i++) {
		int x = read(), y = read(), z = read();
		add(x, y, z);
	}
	dfs(1, 0);
	printf("%lld\n", dp[1][k]);
	return 0;
}
```

