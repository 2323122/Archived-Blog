---
title: Luogu P3959 宝藏 解题报告
categories: 解题报告
toc: true
date: 2019-09-06 19:40:36
tags:
	- 图论
	- DP
	- 状压DP
	- 搜索
	- DFS
top:
thumbnail: https://i.loli.net/2019/09/06/5zxGZeEsHmYbV6o.jpg
---

重边卡了 $60$ 分，这出题人⑧太行。

# 题目链接

[P3959 宝藏](https://www.luogu.org/problem/P3959)

# 题意简述

求给定图的最小代价生成树。

其中代价定义为：$w = \mathrm L \times \mathrm K$

其中 $L$ 为路径长度， $K$ 为已知生成树中该路径起点的深度。

<!--more-->

# 解题思想

首先 $n$ 非常小，我们不难想到把整个图中已经构造出的生成树包含哪些点作为状态压进去。

然后可以基于搜索，枚举一下每次加的边的起点，更新新点的深度，记得要回溯。

最小化就是状压 $\mathrm {DP}$ 的基本操作了，不详细说明。

还有一个坑点就是，**读入是要判重边的**，不加会少 $60$ 分。出题人卡这个真的是闲的蛋疼。

# 参考代码

```c++
#include <cstdio>
#include <cstring>
#include <cctype>

const int N = 15;

inline int read() {
    char ch = getchar(); int r = 0, w = 1;
    while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
    while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
    return r * w;
}

inline int min(int a, int b) {return a < b ? a : b;}

int n, m, ans = 0x7fffffff;
int dp[1 << N], dis[N][N], d[N];
bool cnt[N][N];

inline void add(int x, int y, int z) {
    dis[x][y] = dis[y][x] = z;
    cnt[x][y] = cnt[y][x] = 1;
}

void dfs(int s) {
    for(register int x = 1; x <= n; x++) {
        if((1 << (x - 1)) & s) {
            for(register int y = 1; y <= n; y++) {
                if(cnt[x][y] && !((1 << (y - 1)) & s)) {
                    if(dp[s | (1 << (y - 1))] > dp[s] + dis[x][y] * d[x]) {
                        int tmp = d[y];
                        d[y] = d[x] + 1;
                        dp[s | (1 << (y - 1))] = dp[s] + dis[x][y] * d[x];
                        dfs(s | (1 << (y - 1)));
                        d[y] = tmp;
                    }
                }
            }
        }
    }
}

int main() {
    n = read(), m = read();
    memset(dis, 0x3f, sizeof dis);
    for(register int i = 1; i <= m; i++) {
        int x = read(), y = read(), z = read();
        if(z < dis[x][y]) add(x, y, z);
    }
    for(register int rt = 1; rt <= n; rt++) {
        memset(d, 0x3f, sizeof d);
        for(register int i = 1; i <= (1 << n) - 1; i++) dp[i] = 0x7fffffff;
        dp[1 << (rt - 1)] = 0;
        d[rt] = 1;
        dfs(1 << (rt - 1));
        ans = min(ans, dp[(1 << n) - 1]);
    }
    printf("%d\n", ans);
    return 0;
}
```

