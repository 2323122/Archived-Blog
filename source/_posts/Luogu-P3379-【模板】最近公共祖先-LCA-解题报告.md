---
title: Luogu P3379 【模板】最近公共祖先(LCA) 解题报告
categories: 解题报告
toc: true
date: 2019-09-28 17:30:41
tags:
top:
thumbnail: https://i.loli.net/2019/10/08/avwUQncRr3p7uhG.jpg
---

$20$ 行的 $\mathrm{LCT}$ 了解一下$\sim$。

# 题目链接

[P3379 【模板】最近公共祖先（LCA）](https://www.luogu.org/problem/P3379)

# 题意简述

求 $\mathrm{LCA}$。

<!--more-->

# 解题思想

考虑 $\mathrm{LCT}$ 维护。

具体做法就是，在 `access` 的时候，如果我们返回最后一次虚实链变换时虚边父节点的值，那么接连两次 `access` 的后一次的返回值就是两个节点的 $\mathrm{LCA}$。

由于只需要维护 $\mathrm{LCA}$，所以我们只需要维护一下翻转tag，别的信息不用维护，这样既减少了码量，常数也能适当小一点~~虽然还是那么大。~~

# 参考代码

```c++
#include <bits/stdc++.h>
inline int read() {char ch = getchar(); int r = 0, w = 1; while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();} while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();} return r * w;}
const int N = 5e5 + 6;
int n, m, S, x, y, ch[N][2], fa[N], tag[N], s[N];
void pushdown(int o) {if(tag[o]) tag[o] ^= 1, tag[ch[o][0]] ^= 1, tag[ch[o][1]] ^= 1, std::swap(ch[o][0], ch[o][1]);}
bool get(int o) {return ch[fa[o]][1] == o;}
bool isrt(int o) {return ch[fa[o]][0] != o && ch[fa[o]][1] != o;}
void rotate(int x) {int y = fa[x], z = fa[y], chk = get(x); if(!isrt(y)) ch[z][ch[z][1] == y] = x; ch[y][chk] = ch[x][chk ^ 1], fa[ch[x][chk ^ 1]] = y, ch[x][chk ^ 1] = y, fa[y] = x, fa[x] = z;}
void splay(int x) {int top; s[top = 1] = x; for(register int i = x; !isrt(i); i = fa[i]) s[++top] = fa[i]; while(top) pushdown(s[top--]); for(register int f; f = fa[x], !isrt(x); rotate(x)) {if(!isrt(f)) rotate(get(f) == get(x) ? f : x);}}
int access(int x) {int o; for(o = 0; x; o = x, x = fa[x]) splay(x), ch[x][1] = o; return o;}
void mkrt(int x) {access(x); splay(x); tag[x] ^= 1;}
void link(int x, int y) {mkrt(x); fa[x] = y;}

int main() {
	n = read(), m = read(), S = read();
	for(register int i = 1; i < n; i++) x = read(), y = read(), link(x, y);
	mkrt(S);
	while(m--) x = read(), y = read(), access(x), printf("%d\n", access(y));
	return 0;
}
```

