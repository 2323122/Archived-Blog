---
title: Luogu P4113 [HEOI2012]采花 解题报告
categories: 解题报告
toc: true
date: 2019-09-09 20:50:39
tags:
	- 数据结构
	- 离线
	- 树状数组
top:
thumbnail: https://i.loli.net/2019/09/09/hDMUvRIiCazA1fw.jpg
---

几乎一样的题想不出来解法我太难了。

# 题目链接

[P4113 [HEOI2012]采花](https://www.luogu.org/problem/P4113)

# 题意简述

多次询问一个序列的某个子区间内哪些元素出现了两次以上。

<!--more-->

# 解题思想

考虑离线。。。

我们考虑对答案有贡献的区间，对于某个点的元素 $x$，钦定它已出现过两次以上，那么，如果记它上一次出现的位置为 $last[x]$，上上一次出现的位置为 $llast[x]$，不难得到 $(llast[x],last[x]]$ 这个区间对答案是有贡献的。

然后原序列按位考虑，对每一位枚举右端点为当前位的询问，并处理。

答案输出要按原序。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <algorithm>
#include <iostream>

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

const int N = 2e6 + 5;

int a[N], t[N], last[N], llast[N], ans[N], n, m, c, now = 1;
struct que {int l, r, id;} q[N];
inline int lowbit(int x) {return x & -x;}
inline void modify(int x, int k) {for(; x <= n; x += lowbit(x)) t[x] += k;}
inline int query(int x) {int ans = 0; for(; x; x -= lowbit(x)) ans += t[x]; return ans;}
inline bool cmp(que a, que b) {return a.r < b.r;}

int main() {
	n = read(), c = read(), m = read();
	for(register int i = 1; i <= n; i++) {
		a[i] = read();
	}
	for(register int i = 1; i <= m; i++) {
		q[i].l = read(), q[i].r = read(), q[i].id = i;
	}
	std::sort(q + 1, q + m + 1, cmp);
	for(register int i = 1; i <= n; i++) {
		if(last[a[i]] != llast[a[i]]) {
			if(llast[a[i]]) modify(llast[a[i]], -1);
			if(last[a[i]]) modify(last[a[i]], 1);
		}
		for(; now <= m && q[now].r == i; now++)
			ans[q[now].id] = query(q[now].r) - query(q[now].l - 1);
		llast[a[i]] = last[a[i]];
		last[a[i]] = i;
	}
	for(register int i = 1; i <= m; i++) {
		printf("%d\n", ans[i]);
	}
	return 0;
}
```

