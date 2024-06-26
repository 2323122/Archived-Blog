---
title: UVA1510 Neon Sign 解题报告
categories: 解题报告
date: 2019-04-29 10:05:28
tags:
  - 思维题
  - 组合计数
top: 
---

这种题挺好玩的2333

# 题目链接

[(洛谷)UVA1510 Neon Sign](<https://www.luogu.org/problemnew/show/UVA1510>)

# 题意简述

一个圆上有n个点，告诉你每个点与其它点的边的颜色（红或蓝），求颜色相同的边组成的三角形的个数。

<!--more-->

# 解题思想

考虑**正难则反**，不妨统计存在**不同色的边的三角形**的个数，再用总数减。

统计总数：
$$Sum = C_n^3 = n\times (n - 1)\times (n - 2)/6$$

统计不同色边三角形个数：
不妨先**钦定**两条边颜色一定不同，那么对于**任意颜色**的第三条边，这个三角形**一定**是**不符合条件**的。

考虑每个点，由于每条边被两个点相连，所以我们可以**在读入时**统计每个点连接的蓝边`b[i]`与红边`r[i]`条数，求和后**除以2**.

即：

$$Opp = \frac{1}{2}\sum b[i]\times r[i]$$

那么答案为：
$$Ans = Sum - Opp = n\times (n - 1)\times (n - 2)/6 -  \frac{1}{2}\sum b[i]\times r[i]$$

计算是$O(n)$的，读入是$O(n^2)$的，故**总复杂度**为$O(n^2)$

# 参考代码

```c++
#include <cstdio>
#include <cstring>

int a[1001], b[1001];

int main() {
    int t;
    scanf("%d", &t);
    while(t--) {
        memset(a, 0, sizeof(a));
        memset(b, 0, sizeof(b));
        int n;
        scanf("%d", &n);
        for(register int i = 1; i < n; i++) {
            for(register int j = i + 1; j <= n; j++) {
                int opt;
                scanf("%d", &opt);
                if(opt) a[i]++, a[j]++;
                else b[i]++, b[j]++;
            }
        }
        int sum = 0;
        for(register int i = 1; i <= n; i++) sum += a[i] * b[i];
        printf("%d\n", n * (n - 1) * (n - 2) / 6 - sum / 2); 
    }
    return 0;
}
```

~~十二省联考命题组提醒您：~~

~~数据千万条，清空第一条。~~

~~多测不清空，爆零两行泪。~~

