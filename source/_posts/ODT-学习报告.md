---
title: ODT 学习报告
toc: true
date: 2019-10-12 11:07:18
tags:
	- 数据结构
	- ODT
categories: 学习报告
top:
thumbnail: https://s2.ax1x.com/2019/10/12/uLb8oT.jpg
---

学了这个玄学玩意...相较于珂朵莉树还是感觉叫 ODT 比较舒服呢

~~最近怎么都在学玄学~~

<!--more-->

# 介绍

珂朵莉树是一个基于 `set` 的玄学数据结构..在数据随机的情况下期望复杂度是 $\log$ 的。但是很好卡...

# 主要思想

把性质相同的点绑在一起处理。剩下的操作暴力..

在数据随机的情况下跑的飞快，可以有力地处理区间元素幂求和问题。

# 操作

## 建树

我们将性质相同的节点存为一个区间，以左端点为序塞到 `set` 里。

```c++
struct Node {
    int l, r;
	mutable int k;
    Node(int _l, int _r, int _k) : l(_l), r(_r), k(_k) {}
    bool operator < (const Node &a) const {
        return l < a.l;
    }
};

set <Node> s;
#define It set <Node> :: iterator
```

然后在未进行任何操作的时候，将每个点存成长度为一的区间塞进去。
为了防止某些边界错误，一般会多塞一个空节点 `Node(n + 1, n + 1, 0)`。

一个例子：

```c++
for(register int i = 1; i <= n; i++) {
    s.insert(Node(i, i, a[i]));
}
s.insert(Node(n + 1, n + 1, 0));
```

遍历原序列的话只需要执行以下操作：

```c++
for(It it = s.begin(); it != s.end(); it++) {
    for(register int i = it -> l; i <= it -> r; i++) {
        printf("%d ", it -> k);
    }
}
```



## split(x)

主要操作。用于取出一个点以进行处理。

根据主要思想，我们知道，当我们要单独处理一个点时，它可能由于性质相同被捆绑到无关区间里面，而 `split()` 操作则是将该点所在的区间以该点为界分裂成两段。

具体实现为先在 `set` 中二分出该点的所在区间 $[l, r]$。如果该区间左端点恰好为该点，直接返回即可。否则将原区间从 `set` 中删除，并插入两个性质相同的区间 $[l, x - 1]$ 和 $[x, r]$，并返回后者的头指针。

## assign(l, r, k)

区间推平。即将区间 $[l, r]$ 全部改为性质为 $k$ 的点。只需要 `split` 出两端点，然后将两端点间的所有区间全部删除，并插入一个性质为 $k$ 的大区间 $[l, r]$ 即可。

## 剩下的操作

根据题目要求取出操作区间，考虑每个捆绑区间对操作的贡献暴力处理即可。这里以区间桶排序为例:

```c++
void modify(int l, int r) {
    memset(a, 0, sizeof a); // 清空桶
    It itr = split(r + 1), itl = split(l);
    for(; itl != itr; itl = s.erase(itl)) a[itl -> k] += itl -> r - itl -> l + 1;
    for(register int i = 1; i <= size; i++) if(a[i]) s.insert(Node(l, l + a[i] - 1, i)), l += a[i];
}
```

需要注意的是，分裂出操作区间的时候，由于取出前面的端点会对后面的端点产生影响，所以我们需要先取出操作区间右端点。即：

```c++
It itr = split(r + 1), itl = split(l);
```

# 说明

-   由于复杂度比较玄学...所以一般别用ODT...
-   操作多往区间影响上考虑，不要直接操作全局，这样复杂度会高到爆炸。
-   可以替换树剖的线段树...但是笔者太菜还没写过....

# 例题

## [CF896C Willem, Chtholly and Seniorious](https://www.luogu.org/problem/CF896C)（~~万恶之源~~）

### 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <set>
#include <vector>
#include <algorithm>
#define It set <Node> :: iterator

typedef long long ll;

using std::set;
using std::vector;
using std::pair;

const int N = 1e5 + 6;

struct Node {
	int l, r;
	mutable ll val;
	Node(int _l, int _r = -1, ll _val = 0) : l(_l), r(_r), val(_val) {}
	bool operator < (const Node &a) const {
		return l < a.l;
	}
};

set <Node> s;

It split(int x) {
	It it = s.lower_bound(Node(x));
	if(it != s.end() && it -> l == x) return it;
	it--;
	int l = it -> l, r = it -> r;
	ll k = it -> val;
	s.erase(it);
	s.insert(Node(l, x - 1, k));
	return s.insert(Node(x, r, k)).first;
}

void assign(int l, int r, ll k) {
	It itl = split(l), itr = split(r + 1);
	s.erase(itl, itr);
	s.insert(Node(l, r, k));
}

void add(int l, int r, ll k) {
	It itl = split(l), itr = split(r + 1);
	for(; itl != itr; itl++) itl -> val += k;
}

ll rk(int l, int r, int k) {
	vector < pair <ll, int> > vec;
	It itl = split(l), itr = split(r + 1);
	vec.clear();
	for(; itl != itr; itl++) vec.push_back(pair <ll, int> (itl -> val, itl -> r - itl -> l + 1));
	std::sort(vec.begin(), vec.end());
	for(vector < pair <ll, int> > :: iterator it = vec.begin(); it != vec.end(); it++) {
		k -= it -> second;
		if(k <= 0) return it -> first;
	}
	return -1ll;
}

ll power(ll a, int b, int p) {
	ll res = 1;
	a %= p;
	while(b) {
		if(b & 1) res = res * a % p;
		b >>= 1;
		a = a * a % p;
	}
	return res % p;
}

ll sum(int l, int r, int b, int p) {
	It itl = split(l), itr = split(r + 1);
	ll res = 0;
	for(; itl != itr; itl++) res = (res + (itl -> r - itl -> l + 1) * (power(itl -> val, b, p)) % p) % p;
	return res;
}

int n, m;
ll a[N];
ll vmax, seed;

ll rnd() {
	ll res = seed;
	seed = (seed * 7 + 13) % (1000000007);
	return res;
}

int main() {
	scanf("%d%d%lld%lld", &n, &m, &seed, &vmax);
	for(register int i = 1; i <= n; i++) {
		a[i] = rnd() % vmax + 1;
		s.insert(Node(i, i, a[i]));
	}
	s.insert(Node(n + 1, n + 1, 0));
	for(register int i = 1; i <= m; i++) {
		int op = int(rnd() % 4) + 1, l = int(rnd() % n) + 1, r = int(rnd() % n) + 1;
		if(l > r) std::swap(l, r);
		int x, y;
		if(op == 3) x = int(rnd() % (r - l + 1)) + 1;
		else x = int(rnd() % vmax) + 1;
		if(op == 4) y = int(rnd() % vmax) + 1;
		if(op == 1) add(l, r, ll(x));
		else if(op == 2) assign(l, r, ll(x));
		else if(op == 3) printf("%lld\n", rk(l, r, x));
		else if(op == 4) printf("%lld\n", sum(l, r, x, y));
	}
	return 0;
}
```

## [CF915E Physical Education Lessons](https://www.luogu.org/problem/CF915E)
### 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <algorithm>
#include <set>

#define It set <Node> :: iterator

using std::set;

inline int read() {
	char ch = getchar(); int r = 0, w = 1;
	while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
	while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
	return r * w;
}

struct Node {
	int l, r;
	mutable bool t;
	Node(int _l, int _r = -1, bool _t = 0) : l(_l), r(_r), t(_t) {}
	bool operator < (const Node &a) const {
		return l < a.l;
	}
};

int n, m, sum;
set <Node> s;

It split(int x) {
	It it = s.lower_bound(Node(x));
	if(it != s.end() && it -> l == x) return it;
	it--;
	int l = it -> l, r = it -> r;
	bool t = it -> t;
	s.erase(it);
	s.insert(Node(l, x - 1, t));
	return s.insert(Node(x, r, t)).first;
}

void assign(int l, int r, bool t) {
	It itr = split(r + 1), itl = split(l), it = itl;
	for(; it != itr; it++)
		sum -= it -> t * (it -> r - it -> l + 1);
	s.erase(itl, itr);
	s.insert(Node(l, r, t));
	sum += (r - l + 1) * t;
}

int main() {
	n = read(), m = read(), sum = n;
	s.insert(Node(1, n, 1));
	while(m--) {
		int l = read(), r = read(), t = read() - 1;
		assign(l, r, t);
		printf("%d\n", sum);
	}
	return 0;
}
```

## [P2787 语文1（chin1）- 理理思维](https://www.luogu.org/problem/P2787)

### 参考代码

```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <set>

using std::set;

int n, m;
int a[27];
char S[50020];

struct Node {
	int l, r;
	mutable int k;
	Node(int _l, int _r = -1, int _k = 0) : l(_l), r(_r), k(_k) {}
	bool operator < (const Node &a) const {
		return l < a.l;
	}
};

set <Node> s;

int turn(char c) {
	if(c >= 'a' && c <= 'z') return int(c - 'a' + 1);
	return int(c - 'A' + 1);
}

#define It set <Node> :: iterator

It split(int x) {
	It it = s.lower_bound(Node(x));
	if(it != s.end() && it -> l == x) return it;
	it--;
	int l = it -> l, r = it -> r, k = it -> k;
	s.erase(it);
	s.insert(Node(l, x - 1, k));
	return s.insert(Node(x, r, k)).first;
}

void assign(int l, int r, int k) {
	It itr = split(r + 1), itl = split(l);
	s.erase(itl, itr);
	s.insert(Node(l, r, k));
}

void modify(int l, int r) {
	memset(a, 0, sizeof a);
	It itr = split(r + 1), itl = split(l);
	for(; itl != itr; itl = s.erase(itl)) a[itl -> k] += itl -> r - itl -> l + 1;
	for(register int i = 1; i <= 26; i++) if(a[i]) s.insert(Node(l, l + a[i] - 1, i)), l += a[i];
}

void query(int l, int r, int k) {
	It itr = split(r + 1), itl = split(l);
	int ans = 0;
	for(; itl != itr; itl++) if(itl -> k == k) ans += itl -> r - itl -> l + 1;
	printf("%d\n", ans);
}

int main() {
	scanf("%d%d", &n, &m);
	scanf("%s", S + 1);
	for(register int i = 1; i <= n; i++) s.insert(Node(i, i, turn(S[i])));
	while(m--) {
		int op, l, r, k; char ch;
		scanf("%d %d %d", &op, &l, &r);
		if(op != 3) std::cin >> ch, k = turn(ch);
		if(op == 1) query(l, r, k);
		else if(op == 2) assign(l, r, k);
		else if(op == 3) modify(l, r);
	}
	return 0;
}
```

## [P2572 [SCOI2010\]序列操作](https://www.luogu.org/problem/P2572)

### 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <set>

using std::set;

#define It set <Node> :: iterator

struct Node {
	int l, r;
	mutable bool k;
	Node(int _l, int _r = -1, int _k = 0) : l(_l), r(_r), k(_k) {}
	bool operator < (const Node &a) const {
		return l < a.l;
	}
};

int n, m;
set <Node> s;

It split(int x) {
	It it = s.lower_bound(Node(x));
	if(it != s.end() && it -> l == x) return it;
	it--;
	int l = it -> l, r = it -> r;
	bool k = it -> k;
	s.erase(it);
	s.insert(Node(l, x - 1, k));
	return s.insert(Node(x, r, k)).first;
}

void assign(int l, int r, bool k) {
	It itr = split(r + 1), itl = split(l);
	s.erase(itl, itr);
	s.insert(Node(l, r, k));
}

void modify(int l, int r) {
	It itr = split(r + 1), itl = split(l);
	for(; itl != itr; itl++) itl -> k = !itl -> k;
}

void query1(int l, int r) {
	It itr = split(r + 1), itl = split(l);
	int ans = 0;
	for(; itl != itr; itl++) ans += (itl -> r - itl -> l + 1) * itl -> k;
	printf("%d\n", ans);
}

void query2(int l, int r) {
	It itr = split(r + 1), itl = split(l);
	int tmp = 0, ans = 0;
	for(; itl != itr; itl++) if(itl -> k) tmp += itl -> r - itl -> l + 1, ans = ans > tmp ? ans : tmp; else tmp = 0;
	printf("%d\n", ans);
}

int main() {
	scanf("%d%d", &n, &m);
	for(register int i = 1; i <= n; i++) {
		int x;
		scanf("%d", &x);
		s.insert(Node(i, i, x));
	}
	s.insert(Node(n + 1, n + 1, 0));
	while(m--) {
		int op, l, r;
		scanf("%d%d%d", &op, &l, &r);
		l++, r++;
		if(op == 0) assign(l, r, 0);
		else if(op == 1) assign(l, r, 1);
		else if(op == 2) modify(l, r);
		else if(op == 3) query1(l, r);
		else query2(l, r);
	}
	return 0;
}
```

