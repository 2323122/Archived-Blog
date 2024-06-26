---
title: Luogu P1396 营救 解题报告
categories: 解题报告
date: 2019-04-18 23:01:06
tags:
	- 图论
	- MST
top: 
---

## 题意简述
给你一张图，有边权，求使得源点汇点联通的路径中边最大值的最小值。（貌似还是不很清楚）
<!--more-->
## 思想
看到最大的最小很多人想到二分，但是可以有更好的解法。
我们不妨用**并查集**维护这个图，将边从小到大排序，每次取出边权最小的边，若该边的起点与终点未在一个集合内，就将其合并。当源点与汇点在一个集合内时，当前边的权值就是答案。
## 并查集基本操作
这里有一个~~比较优美的~~路径压缩的实现：
```cpp
int find(int x) {return fa[x] = fa[x] == x ? x : find(fa[x]);}
```
一行代码。
**合并**的话在主函数内直接写就行了，没有一些附加消息没必要单独写函数。

## Code
40行代码（有快读）
```c++
#include <cstdio>
#include <cctype>
#include <algorithm>

inline int read() {
    char ch = getchar(); int r = 0;
    while(!isdigit(ch)) ch = getchar();
    while(isdigit(ch)) r = r * 10 + ch - '0', ch = getchar();
    return r;
}

const int N = 20020;

struct edge {
    int x, y, z;
} e[N];

int fa[N];

inline bool cmp(edge a, edge b) {return a.z < b.z;}

int find(int x) {return fa[x] = fa[x] == x ? x : find(fa[x]);}

int main() {
    int n = read(), m = read(), s = read(), t = read();
    for(register int i = 1; i <= n; i++) fa[i] = i;
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read(), z = read();
        e[i] = (edge) {x, y, z};
    }
    std::sort(e + 1, e + m + 1, cmp);
    for(register int i = 1; i <= m; i++) {
        int x = find(e[i].x), y = find(e[i].y);
        if(x != y) fa[x] = y;
        if(find(s) == find(t)) {
            printf("%d", e[i].z);
            return 0;
        }
    }
}
```

