---
title: Luogu P1972 [SDOI2009]HH的项链 解题报告
categories: 解题报告
toc: true
date: 2019-09-09 11:25:21
tags:
	- 数据结构
	- 离线
	- 树状数组
top:
thumbnail: https://i.loli.net/2019/09/09/b5HDrElg9kFNLvx.jpg
---

~~面向数据编程~~

# 题目链接

[P1972 [SDOI2009]HH的项链](https://www.luogu.org/problem/P1972)

# 题意简述

多次询问序列中某区间不同元素的个数。

<!--more-->

# 解题思想

答案可以理解为仅被区间后面的几个元素贡献，前面的重复元素可以全部替换为 $0$。

考虑离线，为了消除询问间的影响，我们将询问按右端点排序，然后依次考虑每个询问，使用树状数组维护一个区间内不同元素的个数，即出现一个不同元素时，直接在序列**位置** $+1$，同时维护每一个元素上次出现的位置，若出现相同元素，先在上次出现的位置 $-1$，消除相同元素重复贡献，然后再在该位置 $+1$，同时更新这个元素上次出现的位置为**当前位置**。

在读入询问时要记录询问的顺序，因为排序会导致其改变。

输出时按读入顺序输出答案。

**注意，树状数组操作的一定是位置而不是元素！！**

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <algorithm>
#define int long long

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

const int N = 1000010;

int n, m;
int a[N], t[N], last[N], ans[N];
inline int lowbit(int x) {return x & -x;}
inline void add(int x, int k) {for(; x <= n; x += lowbit(x)) t[x] += k;}
inline int query(int x) {int ans = 0; for(; x; x -= lowbit(x)) ans += t[x]; return ans;}

struct que {
	int l, r, id;
} q[N];

inline bool cmp(que a, que b) {return a.r < b.r;}

signed main() {
	n = read();
	for(register int i = 1; i <= n; i++) {
		a[i] = read();
	}
	m = read();
	for(register int i = 1; i <= m; i++) {
		q[i].l = read(), q[i].r = read(), q[i].id = i;
	}
	std::sort(q + 1, q + m + 1, cmp);
	int l = 1;
	for(register int i = 1; i <= m; i++) {
		for(register int j = l; j <= q[i].r; j++) {
			if(last[a[j]]) add(last[a[j]], -1);
			add(j, 1);
			last[a[j]] = j;
		}
		l = q[i].r + 1;
		ans[q[i].id] = query(q[i].r) - query(q[i].l - 1);
	}
	for(register int i = 1; i <= m; i++) {
		printf("%lld\n", ans[i]);
	}
	return 0;
}
```

