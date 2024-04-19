---
title: Luogu P3953 逛公园 解题报告
categories: 解题报告
toc: true
date: 2019-09-19 16:45:16
tags:
	- 图论
	- 最短路
	- 计数
	- DP
top:
thumbnail: https://i.loli.net/2019/09/19/tJfnkZWjQCcEpDw.jpg
---

交题不删 `fprintf(stderr)` 的我事屑。

# 题目链接

[P3953 逛公园](https://www.luogu.org/problem/P3953)

# 题意简述

给你一张图，记两点间最短路为 $dis(i, j)$ ，求点 $1$ 到点 $n$ 长度不超过 $dis(1, n) + k$ 的路径的数量。

<!--more-->

# 解题思想

先反图跑 `Dijkstra`，求出点 $n$ 到各点的最短路的长度，记为 $dis(n, i)$。

考虑 `DP`，对于点 $x$，我们记 $dp[x][k]$ 为在点 $x$ ，且可用冗余长度还剩下 $k$ 时的方案数。

那么显然有 $dp[x][flow] = \sum\limits_{<x, y>\in E}dp[y][rest]$ ，其中 $rest = dis(y, n) + len(x, y) - dis (x, n) \in[0, k]$。

记忆化搜索实现即可。

但是有一种特殊的情况需要考虑：
若被计数路径上出现 $0$ 环，那么可行路径有无数条，记搜的时候特判输出 $-1$ 即可。

~~有点 `Dinic` 内味儿了。~~

~~`fprintf()` 是真的慢。~~

# 参考代码

```c++
#pragma GCC optimize(2)
#include <cstdio>
#include <cctype>
#include <cstring>
#include <queue>
#define getchar() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 21, stdin), p1 == p2) ? EOF : *p1++)

char buf[1 << 21], *p1 = buf, *p2 = buf;

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

const int N = 100010;
const int M = N << 2;
const int K = 50 + 6;

int n, m, k, p, ans;
int head[2][N], ver[2][M], nxt[2][M], edge[2][M], cnt;
int dp[N][K], d[N];
bool v[N][K], flg;

inline void add(int x, int y, int z) {
	ver[0][++cnt] = y, edge[0][cnt] = z, nxt[0][cnt] = head[0][x], head[0][x] = cnt;
	ver[1][cnt] = x, edge[1][cnt] = z, nxt[1][cnt] = head[1][y], head[1][y] = cnt;
}

struct data {
	int pos, dis;
	data(int pos, int dis) {this -> pos = pos, this -> dis = dis;}
	bool operator < (const data &a) const {
		return dis > a.dis;
	}
};

std::priority_queue <data> heap;

inline void dijkstra(int s) {
	memset(d, 0x3f, sizeof d);
	heap.push(data(s, d[s] = 0));
	while(!heap.empty()) {
		data tmp = heap.top();
		heap.pop();
		if(tmp.dis != d[tmp.pos]) continue;
		for(register int i = head[1][tmp.pos]; i; i = nxt[1][i])
			if(d[ver[1][i]] > d[tmp.pos] + edge[1][i])
				heap.push(data(ver[1][i], d[ver[1][i]] = tmp.dis + edge[1][i]));
	}
}

inline void init() {memset(head, 0, sizeof head); memset(dp, 0, sizeof dp); cnt = ans = flg = 0;}

int dfs(int x, int flow) {
	if(v[x][flow]) return flg = 1, 0;
	if(dp[x][flow] > 0) return dp[x][flow];
	v[x][flow] = 1;
	int sum = 0;
	for(register int i = head[0][x]; i; i = nxt[0][i]) {
		int y = ver[0][i], z = edge[0][i];
		int rest = flow - (d[y] + z - d[x]);
		if(rest < 0 || rest > k) continue;
		sum = (sum + dfs(y, rest)) % p;
		if(flg) return 0;
	}
	if(x == n && flow == 0) sum = 1;
	v[x][flow] = 0;
	return dp[x][flow] = sum;
}

int main() {
	int t = read();
	while(t--) {
		init();
		n = read(), m = read(), k = read(), p = read();
		for(register int i = 1; i <= m; i++) {
			int x = read(), y = read(), z = read();
			add(x, y, z);
		}
		dijkstra(n);
		for(register int i = 0; i <= k; i++) {
			memset(v, 0, sizeof v);
			ans = (ans + dfs(1, i)) % p;
		}
		printf("%d\n", flg ? -1 : ans);
	}
	return 0;
}
```

****