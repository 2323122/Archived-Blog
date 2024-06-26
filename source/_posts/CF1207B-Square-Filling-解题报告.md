---
title: CF1207B Square Filling 解题报告
categories: 解题报告
toc: true
date: 2019-08-25 18:20:09
tags:
    - 贪心
    - 模拟
top: 
thumbnail: https://i.loli.net/2019/08/25/fY9JvNFrk7lzUuZ.jpg
---

考场上想了个很$naive$的`dp`，$\color{black}{Q}\color{red}{uank}$大爷提醒后才发现没有最优化。

身败名裂

# 题目链接

[(Luogu)CF1207B Square Filling](https://www.luogu.org/problem/CF1207B)

# 题意简述

给你一个`01`矩阵，每次操作可将一个$2\times 2$的正方形全部赋值为$1$，问你能不能将一个空矩阵变成目标矩阵。

<!--more-->

# 解题思想


既然题目中不要求最优解，我们就不需要考虑少覆盖几个正方形。

直接从$(1, 1)$开始枚举，依次考虑以当前点为**左上角**的正方形，如果可以覆盖，直接覆盖并记录路径。

然后在全部覆盖完后将答案矩阵和目标矩阵进行对比，若一致则输出方案。

注意不要越界。

# 参考代码

```c++
#include <cstdio>

const int N = 100;

int a[N][N], b[N][N];

int n, m;

int x[N * N], y[N * N], cnt;

int main() {
    scanf("%d%d", &n, &m);
    for(register int i = 1; i <= n; i++) {
        for(register int j = 1; j <= m; j++) {
            scanf("%d", &a[i][j]);
        }
    }
    for(register int i = 1; i <= n; i++) {
        for(register int j = 1; j <= m; j++) {
            if(a[i][j] && a[i + 1][j] && a[i][j + 1] && a[i + 1][j + 1] && i + 1 <= n && j + 1 <= m)
                b[i][j] = b[i + 1][j] = b[i][j + 1] = b[i + 1][j + 1] = 1, x[++cnt] = i, y[cnt] = j;
        }
    }
    for(register int i = 1; i <= n; i++)
        for(register int j = 1; j <= m; j++)
            if(a[i][j] != b[i][j])
                return puts("-1"), 0;
    printf("%d\n", cnt);
    for(register int i = 1; i <= cnt; i++) {
        printf("%d %d\n", x[i], y[i]);
    }
    return 0;
}
```