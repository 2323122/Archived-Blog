---
title: Luogu P3623 [APIO2008]免费道路 解题报告
categories: 解题报告
date: 2019-08-03 11:17:59
tags:
	- 图论
	- MST
	- 贪心
top:
---

生成树太蔡了来练一练。

# 题目链接

[P3623 [APIO2008]免费道路](https://www.luogu.org/problem/P3623)

# 题意简述

给你一张图，边权为$0$或$1$，求权值为$k$的生成树。

<!--more-->

# 解题思想

我们先把边权为0的边优先加入生成树，然后可以得到一些边权为1的必须加入生成树的边，对它们打上标记。

然后再把边权为1的边优先加入生成树，并且要优先加入有标记的边，然后当边权为1的边加到$k$条时就跳到边权为0的边加完剩下的生成树就行了。

判定无解需要考虑生成树权值是否为k和是否构成生成树。

注意题目中读入的0和1是反的...

要求输出的0和1也是反的...

# 参考代码

```c++
#include <cstdio>
#include <algorithm>

using std::sort;

const int N = 2e4 + 5;
const int M = 1e5 + 6;

int n, m, k, cnt;
int fa[N];

int find(int x) {return fa[x] == x ? x : fa[x] = find(fa[x]);}

struct edge {
    int x, y, z, mark;
} e[M];

inline bool cmp1(edge a, edge b) {
    return a.z < b.z;
}

inline bool cmp2(edge a, edge b) {
    if(a.z == b.z) return a.mark > b.mark;
    return a.z > b.z;
}

int main() {
    scanf("%d%d%d", &n, &m, &k);
    for(register int i = 1; i <= m; i++) {
        int x, y, z;
        scanf("%d%d%d", &x, &y, &z);
        e[i] = edge{x, y, 1 - z};
    }
    for(register int i = 1; i <= n; i++) fa[i] = i;
    sort(e + 1, e + m + 1, cmp1);
    for(register int i = 1; i <= m; i++) {
        int x = e[i].x, y = e[i].y;
        if(find(x) == find(y)) continue;
        fa[find(x)] = find(y), cnt++;
        if(e[i].z) e[i].mark = 1;
        if(cnt == n - 1) break;
    }
    cnt = 0;
    sort(e + 1, e + m + 1, cmp2);
    for(register int i = 1; i <= n; i++) fa[i] = i;
    for(register int i = 1; i <= m; i++) {
        if(e[i].mark) {fa[find(e[i].x)] = find(e[i].y); cnt++, k--; continue;}
        if(find(e[i].x) == find(e[i].y)) continue;
        fa[find(e[i].x)] = find(e[i].y);
        cnt++;
        e[i].mark = 1;
        if(e[i].z) k--;
        if(!k) for(; e[i].z; i++);
        if(cnt == n - 1) break;
    }
    if(k) return puts("no solution"), 0;
    for(register int i = 1; i <= m; i++) {
        if(e[i].mark) k++;
    }
    if(k < n - 1) return puts("no solution"), 0;
    for(register int i = 1; i <= m; i++) {
        if(e[i].mark) printf("%d %d %d\n", e[i].x, e[i].y, 1 - e[i].z);
    }
    return 0;
}
```

