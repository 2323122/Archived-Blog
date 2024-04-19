---
title: Luogu P2325 [SCOI2005]王室联邦 解题报告
categories: 解题报告
toc: true
date: 2019-09-17 11:16:00
tags:
	- 数据结构
	- 图论
	- 分块
	- 树上问题
top:
thumbnail: https://i.loli.net/2019/09/17/FuIYfO2omE3K58q.png
---

感觉一万年没更博客了来水一篇题解。

# 题目链接

[P2325 [SCOI2005]王室联邦](https://www.luogu.org/problem/P2325)

# 题意简述

<!--more-->

给你一颗树，将其分成大小 $\in[B, 3B]$ 的块。

# 解题思想

$\text{dfs}$ 根节点，建立一个栈，对于每个子树递归分块，并在回溯时将节点入栈。若当前新增节点数大于 $B$，就把新增节点分为一个块。

题目中的 “省会” 即为最后一个被分入当前块的点。

块大小大于 $B$ 是显然的。

分类讨论一下未分块节点数量可以得到块大小的上界是 $2B - 1$。

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

const int N = 1010;
const int M = N << 1;

int n;
int fa[N], head[N], ver[M], nxt[M], key[N], pos[N], cnt;
int stk[N], top, tot, b;

inline void add(int x, int y) {
    ver[++cnt] = y, nxt[cnt] = head[x], head[x] = cnt;
    ver[++cnt] = x, nxt[cnt] = head[y], head[y] = cnt;
}

void dfs(int x) {
    int now = top;
    for(register int i = head[x]; i; i = nxt[i]) {
        int y = ver[i];
        if(y == fa[x]) continue;
        fa[y] = x;
        dfs(y);
        if(top - now >= b) {
            key[++tot] = x;
            while(top > now) pos[stk[top--]] = tot;
        }
    }
    stk[++top] = x;
}

int main() {
    n = read(), b = read();
    for(register int i = 1; i < n; i++) add(read(), read());
    dfs(1);
    if(!tot) tot++;
    key[tot] = 1;
    while(top) pos[stk[top--]] = tot;
    printf("%d\n", tot);
    for(register int i = 1; i <= n; i++) printf("%d ", pos[i]);
    puts("");
    for(register int i = 1; i <= tot; i++) printf("%d ", key[i]);
    return 0;
}
```

