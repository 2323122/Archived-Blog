---
title: 动态DP 学习报告
toc: true
date: 2019-09-26 15:00:57
tags:
	- DP
	- DDP
categories: 学习报告
top:
thumbnail: https://i.loli.net/2019/09/26/u2oSXEezlfsHt4w.jpg
---

# 引入

考虑一个问题：
求树上最大权独立集的权值大小。其中点权可更改。

<!--more-->

## 不带修情况

先考虑**不带修**的情况。很简单，直接**树形 DP **求就行了。

设 $f_{i,0}$ 表示点 $i$ **不被选择**时的答案，$f_{i_,1}$ 表示点 $i$ **被选择**时的答案。

那么状态转移方程为：

$$\begin{cases}
f_{i,0} = \sum\limits_{j\in \mathrm{son_i}} \max({f_{j, 0}, f_{j, 1}}) \\\ f_{i, 1} = w_i + \sum\limits_{j\in\mathrm{son_i}}f_{j, 0} 
\end{cases}$$

答案为：$\max(f_{rt_,0}, f_{rt, 1})$。

## 带修情况

我们对该树进行**树链剖分**。

设 $g_{i, 0}$ 表示点 $i$ 及其**重儿子**都不被选择时的答案，$g_{i, 1}$ 表示点 $i$ 被选择、点 $i$ 的重儿子不被选择时的答案。

**$\mathrm{son_i}$ 定义变为点 $i$ 的重儿子。**

由 $g_{i, 0/1}$ 向 $f_{i, 0/1}$ 转移，我们有：

$$\begin{cases} f_{i, 0} = g_{i, 0} + \max(f_{\mathrm{son_i}, 0}, f_{\mathrm{son_i}, 1}) \\\ f_{i, 1} = g_{i, 1} + f_{\mathrm{son_i}, 0} \end{cases}$$

答案为：$\max(f_{rt, 0}, f_{rt, 1})$。

### 广义矩乘

重新定义**矩阵乘法** $A \times B = C$ 为 $C_{i, j} = \max\limits_{k = 1} ^ n (A_{i, k} + B_{k, j})$。

那么上面的**状态转移方程**可以写成：

$$\begin{bmatrix} g_{i, 0} & g_{i, 0} \\\ g_{i, 1} & -\infty \end{bmatrix} \times \begin{bmatrix}f_{\mathrm{son_i}, 0} \\\ f_{\mathrm{son_i}, 1} \end{bmatrix} = \begin{bmatrix}f_{i, 0} \\\ f_{i, 1} \end{bmatrix}$$

所以当我们**修改**了点 $i$ 的点权时，只需要修改 $g_{i, 1}$ 和点 $i$ 到 $rt$ 的每一条重链。

# 算法过程

-   首先，可以在对原树**树剖**的**同时**进行第一遍**树形 DP** ，求出 $f$ 与 $g$ 的初值。
-   然后对每一条**重链**建**线段树**维护一下**区间广义矩乘**。
-   修改的话直接修改 $g_{i, 1}$ 和**线段树**中 $i$ 节点的矩阵。
-   然后需要计算**链顶矩阵**的变化量，并且**上传到它的父节点**。
-   查询输出根到所在重链的链尾矩阵的区间乘就行了。

如果用 LCT 维护的话，改哪个点 `access` 哪个点，不用来回跳重链，比较方便。~~但是我懒得写 LCT。~~

# 参考代码

以[Luogu P4719 【模板】动态 DP](https://www.luogu.org/problem/P4719)为例。

```c++
#include <cstdio>
#include <cctype>
#include <cstring>

#define LL long long

inline LL read() {
	char ch = getchar(); LL r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

template <typename T> inline T max(T a, T b) {return a > b ? a : b;}
template <typename T> inline T min(T a, T b) {return a < b ? a : b;}

const int N = 5e5 + 6;
const int M = N << 1;
const int INF = 0x3f3f3f3f;

int n, m;
int head[N], ver[M], nxt[M], cnt;
int d[N], sz[N], tp[N], ed[N], fa[N], son[N], ptn[N], dfn[N], idx;
int a[N], f[N][2];

struct Matrix {
	int g[2][2];
	Matrix() {memset(g, 0, sizeof g);}
	Matrix operator * (const Matrix &a) const {
		Matrix res;
		for(register int i = 0; i <= 1; i++)
			for(register int j = 0; j <= 1; j++)
				for(register int k = 0; k <= 1; k++)
					res.g[i][j] = max(res.g[i][j], g[i][k] + a.g[k][j]);
		return res;
	}
} t[N], g[N];

inline void add(int x, int y) {
	ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
}

void dfs1(int x) {
	sz[x] = 1;
	f[x][1] = a[x];
	for(register int i = head[x]; i; i = nxt[i]) {
		int y = ver[i];
		if(y == fa[x]) continue;
		fa[y] = x;
		d[y] = d[x] + 1;
		dfs1(y);
		sz[y] += sz[x];
		if(sz[y] > sz[son[x]]) son[x] = y;
		f[x][1] += f[y][0];
		f[x][0] += max(f[y][0], f[y][1]);
	}
}

void dfs2(int x, int top) {
	ptn[dfn[x] = ++idx] = x;
	tp[x] = top;
	ed[top] = idx;
	g[x].g[1][0] = a[x];
	g[x].g[1][1] = -INF;
	if(son[x]) dfs2(son[x], top);
	for(register int i = head[x]; i; i = nxt[i]) {
		int y = ver[i];
		if(y == fa[x] || y == son[x]) continue;
		dfs2(y, y);
		g[x].g[0][0] += max(f[y][0], f[y][1]);
		g[x].g[1][0] += f[y][0];
	}
	g[x].g[0][1] = g[x].g[0][0];
}

#define ls (o << 1)
#define rs (o << 1 | 1)
#define mid ((l + r) >> 1)
#define upd(o) t[o] = t[ls] * t[rs]

void build(int o, int l, int r) {
	if(l == r) {t[o] = g[ptn[l]]; return;}
	build(ls, l, mid);
	build(rs, mid + 1, r);
	upd(o);
}

Matrix query(int o, int l, int r, int ll, int rr) {
	if(ll <= l && rr >= r) return t[o];
	if(rr <= mid) return query(ls, l, mid, ll, rr);
	if(ll > mid) return query(rs, mid + 1, r, ll, rr);
	return query(ls, l, mid, ll, rr) * query(rs, mid + 1, r, ll, rr);
}

void modify(int o, int l, int r, int x) {
	if(l == r) {t[o] = g[ptn[x]]; return;}
	if(x <= mid) modify(ls, l, mid, x);
	else modify(rs, mid + 1, r, x);
	upd(o);
}

void modify(int x, int k) {
	g[x].g[1][0] += k - a[x];
	a[x] = k;
	while(x) {
		Matrix las = query(1, 1, n, dfn[tp[x]], ed[tp[x]]);
		modify(1, 1, n, dfn[x]);
		Matrix now = query(1, 1, n, dfn[tp[x]], ed[tp[x]]);
		x = fa[tp[x]];
		g[x].g[0][0] += max(now.g[0][0], now.g[1][0]) - max(las.g[0][0], las.g[1][0]);
		g[x].g[0][1] = g[x].g[0][0];
		g[x].g[1][0] += now.g[0][0] - las.g[0][0];
	}
}

int main() {
	n = read(), m = read();
	for(register int i = 1; i <= n; i++) a[i] = read();
	for(register int i = 1; i < n; i++) {
		int x = read(), y = read();
		add(x, y);
		add(y, x);
	}
	d[1] = 1;
	dfs1(1);
	dfs2(1, 1);
	build(1, 1, n);
	for(register int i = 1; i <= m; i++) {
		int x = read(), k = read();
		modify(x, k);
		Matrix ans = query(1, 1, n, 1, ed[1]);
		printf("%d\n", max(ans.g[0][0], ans.g[1][0]));
	}
	return 0;
}
```

