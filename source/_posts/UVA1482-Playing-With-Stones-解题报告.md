---
title: UVA1482 Playing With Stones 解题报告
categories: 解题报告
date: 2019-06-07 10:15:08
tags:
  - 博弈论
top:
---

不用SG函数的博弈题不是好博弈题！

# 题目链接

[(洛谷)UVA1482 Playing With Stones](https://www.luogu.org/problemnew/show/UVA1482)

# 题意简述

在每一个的回合内，他可以拿走至少一个石子，但不能超过石子所在堆的石子总数的一半。不能继续操作的游戏者输。给定局面求是否**先手必胜**。

<!--more-->

# 解题思想

类似于$Nim$游戏，我们尝试把**SG**函数搞出来。

手玩一下**堆数为1**的情况，可以发现SG函数的一个规律：

- 当$n$为偶数时，$SG(n) = \frac{n}{2}$
- 当$n$为奇数时，$SG(n) = SG(n / 2)$

不难递归实现**SG**函数：
`int sg(int x) {return x % 2 == 0 ? x / 2 : sg(x / 2);}`

讲一讲**优化方法**：
**奇数偶数判定**及**整数除以2**都可以使用**位运算**加速。

整数除以2就不讲了，大概大家都知道。

对于奇数偶数判定，我们考虑一个数的**二进制表示**。

可以发现，一个数若是**偶数**，那么它的二进制表示下的**最低位一定为0**。

而**奇数**相反。

我们可以使用`x & 1`取出x的最低位，然后判断是否为1就行了。

这是优化后的**SG**函数：

`int sg(int x) {return x & 1 ? sg(x >> 1) : x >> 1;}`

类似于$Nim$博弈，我们将每一堆的SG函数异或和一下，然后判断是否为0就可以得到解。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#define int long long

inline int read() {
    char ch = getchar(); int r = 0, w = 1;
    while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
    while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
    return r * w;
}

int sg(int x) {return x & 1 ? sg(x >> 1) : x >> 1;}

signed main() {
    int t = read();
    while(t--) {
        register int n = read();
        register int a, ans = 0;
        for(register int i = 1; i <= n; i++) a = read(), ans ^= sg(a);
        if(ans) puts("YES");
        else puts("NO");
    }
    return 0;
}
```