---
title: Luogu P1373 小a和uim之大逃离 解题报告
categories: 解题报告
date: 2019-06-11 16:12:36
tags:
  - DP
  - 递推
top:
---

# 题目链接

[**P1373** 小a和uim之大逃离](https://www.luogu.org/problemnew/show/P1373)

# 题意简述

~~感觉题目描述挺清楚的~~

给定矩阵，两个人以任意点为起点，每次可以向下走或向右走，同时两人的得分**交替**加上当前格的值并对$k + 1$取模，可以在任意点由**后者**结束。求两人得分一致的方法数。

<!--more-->

# 解题思想

题目中的方案数与两人的**具体得分**无关，故我们考虑将每一步**两人的差值**作为状态。剩下的状态显而易见：当前位置和取数人。即$dp[x][y][\Delta][op]$表示在坐标为$(x, y)$的方格上由$op$取数，**取完后**两人的差值为$\Delta$

因为由后者结束，所以我们只统计$op = 0$时的答案，并累加。再加上题目中给出的移动方式，我们不难得出状态转移方程：

$$dp[x][y][\Delta][0] = dp[x - 1][y][\Delta - a[x][y]][1] + dp[x][y - 1][\Delta - a[x][y]][1]$$

$$dp[x][y][\Delta][1] = dp[x - 1][y][\Delta + a[x][y]][0] + dp[x][y - 1][\Delta + a[x][y]][0]$$

需要考虑差值小于0的情况，这时将插值替代为模$k + 1$意义下的等价元就行了。

# 参考代码

```c++
#include <cstdio>

const int N = 801;
const int MOD = 1000000007;

int n, m, k, ans;
int a[N][N];
int dp[N][N][17][2];

int main() {
    scanf("%d%d%d", &n, &m, &k);
    k++;
    for(register int i = 1; i <= n; i++) {
        for(register int j = 1; j <= m; j++) {
            scanf("%d", &a[i][j]);
            dp[i][j][a[i][j] % k][0] = 1;
        }
    }
    for(register int i = 1; i <= n; i++) {
        for(register int j = 1; j <= m; j++) {
            for(register int del = 0; del <= k; del++) {
                dp[i][j][del][0] = (dp[i][j][del][0] + dp[i - 1][j][(del - a[i][j] + k) % k][1]) % MOD;
                dp[i][j][del][0] = (dp[i][j][del][0] + dp[i][j - 1][(del - a[i][j] + k) % k][1]) % MOD;
                dp[i][j][del][1] = (dp[i][j][del][1] + dp[i - 1][j][(del + a[i][j] + k) % k][0]) % MOD;
                dp[i][j][del][1] = (dp[i][j][del][1] + dp[i][j - 1][(del + a[i][j] + k) % k][0]) % MOD;
            }
        }
    }
    for(register int i = 1; i <= n; i++) {
        for(register int j = 1; j <= m; j++) {
            ans = (ans + dp[i][j][0][1]) % MOD;
        }
    }
    printf("%d", ans);
    return 0;
}
```

