---
title: POJ 2631 Roads in the North 解题报告
categories: 解题报告
date: 2019-06-04 17:41:35
tags: 
  - 树上问题
  - 树的直径
top:
---

~~我没测，一次过了，有什么好说的。~~

# 题目链接

[POJ 2631 Roads in the North](<http://poj.org/problem?id=2631>)

# 题意简述

给你一棵树，求距离最长的两个点。

<!--more-->

# 解题思想

首先要~~一眼~~看出题目中的图是**一棵树**。然后距离最长的两个点就是**树的直径**。

可以先**任意节点为源**`bfs`一次，求出到该点**距离最长**的一个点。再以它为源`bfs`一次，求出**到该点距离最长**的一个点。那么这**两个点之间的距离**就是**这棵树的直径**。

读入的时候需要记录一下**最小点数**和**最大点数**。

# 参考代码

```c++
//Junk POJ Doesn't Allow me to use <bits/stdc++.h>
#include <cstdio>
#include <queue>
#include <cstring>
using namespace std;

const int N = 10010;
const int M = N << 1;

int head[N], nxt[M], edge[M], ver[M], cnt = 1;
int n, m, s = N, t, l, r;
int d[N], v[N];

inline void add(int x, int y, int z) {
    ver[++cnt] = y, edge[cnt] = z, nxt[cnt] = head[x], head[x] = cnt;
}

int main() {
    int x, y, z;
    while(~scanf("%d %d %d", &x, &y, &z)) {
        add(x, y, z);
        add(y, x, z);
        s = min(s, x);
        s = min(s, y);
        t = max(t, x);
        t = max(t, y);
    }
    queue <int> q;
    q.push(s);
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = 1;
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i], z = edge[i];
            if(v[y]) continue;
            d[y] = d[x] + z;
            q.push(y);
        }
    }
    for(register int i = s; i <= t; i++) {
        if(d[i] > d[r]) r = i;
    }
    q.push(r);
    memset(v, 0, sizeof(v));
    memset(d, 0, sizeof(d));
    while(!q.empty()) {
        int x = q.front();
        q.pop();
        v[x] = 1;
        for(register int i = head[x]; i; i = nxt[i]) {
            int y = ver[i], z = edge[i];
            if(v[y]) continue;
            d[y] = d[x] + z;
            q.push(y);
        }
    }
    for(register int i = s; i <= t; i++) {
        if(d[i] > d[l]) l = i;
    }
    printf("%d", d[l]);
    return 0;
}
```

