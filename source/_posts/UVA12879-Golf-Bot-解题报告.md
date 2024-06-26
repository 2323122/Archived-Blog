---
title: UVA12879 Golf Bot 解题报告
categories: 解题报告
date: 2019-05-18 14:50:53
tags: 
  - 多项式
  - FFT
top:
---

~~大概是最优解~~。

# 题目链接

[(洛谷)UVA12879 Golf Bot](https://www.luogu.org/problemnew/show/UVA12879)

# 题意简述

给你$n$个数，记为数组$A$，再给你$m$个数，记为数组$B$，求从$A$中任选两个数（可重复）相加至多可以得到多少个$B$中的数。

<!--more-->

# 解题思想

~~首先你得看懂题~~，很容易想到一个暴力思路，$O(n^2)$枚举$A$中的元素，然后相加，将得到的数打标记。再$O(n)$枚举$B$中的元素，判断有没有标记。总复杂度$O(n^2)$。

**但是T了。**

**但是T了。**

**但是T了。**

很显然我们需要一个更快的算法。考虑**单项式乘法**，两个单项式相乘，底数不变，指数**相加**。这启示我们把$A$数组元素放到**指数**上。我们不妨根据$A$数组的元素**构造**两个**相同**的多项式，确保每一个**以$A$中元素为指数**的项**系数不为0**。注意：**其他项系数必须为0！**。然后将这两个**多项式相乘**，就可以得到一个**以符合题意的元素为指数的项的系数不为0，其余项系数都为0**的多项式，将该数组与$B$数组比对一下统计答案就行了。

有人会问：多项式乘法不是$O(n^2)$的吗？

我们有**FFT**啊。

乘法部分用**FFT**搞一下就行了。

总时间复杂度$O(n\log n)$。

# 参考代码

```c++
#include <cstdio>
#include <cctype>
#include <cmath>

#define max(a, b) a > b ? a : b

namespace FastIO {
    inline int read() {
        char ch = getchar(); int r = 0, w = 1;
        while(!isdigit(ch)) {if(ch == '-') w = -1; ch = getchar();}
        while(isdigit(ch)) {r = r * 10 + ch - '0', ch = getchar();}
        return r * w;
    }
    void _write(int x) {
        if(x < 0) putchar('-'), x = -x;
        if(x > 9) _write(x / 10);
        putchar(x % 10 + '0');
    }
    inline void write(int x) {
        _write(x);
        puts("");
    }
}

using namespace FastIO;

const int N = 1e6 + 6;
const double Pi = acos(-1.0);

struct Complex {
    double x, y;
    Complex(double _x = 0, double _y = 0) {x = _x, y = _y;}
};

Complex operator + (const Complex &a, const Complex &b) {return Complex(a.x + b.x, a.y + b.y);}
Complex operator - (const Complex &a, const Complex &b) {return Complex(a.x - b.x, a.y - b.y);}
Complex operator * (const Complex &a, const Complex &b) {return Complex(a.x *b.x - a.y * b.y, a.x * b.y + a.y * b.x);}

int ans, limit, tot, n, m;
int r[N], B[N];
Complex A[N];

inline void swap(Complex &a, Complex &b) {Complex tmp = a; a = b, b = tmp;}

inline void FFT(Complex *A, int type) {
    for(register int i = 0; i < limit; i++) {
        if(i < r[i]) swap(A[i], A[r[i]]);
    }
    for(register int mid = 1; mid < limit; mid <<= 1) {
        Complex Wn(cos(Pi / mid), type * sin(Pi / mid));
        for(register int R = mid << 1, j = 0; j < limit; j += R) {
            Complex w(1, 0);
            for(register int k = 0; k < mid; k++, w = w * Wn) {
                Complex x = A[j + k], y = w * A[j + mid + k];
                A[j + k] = x + y;
                A[j + mid + k] = x - y;
            }
        }
    }
}

inline void init() {
    for(register int i = 0; i < limit; i++) A[i].x = A[i].y = r[i] = 0;
    ans = 0; limit = 1; tot = 0;
}

int main() {
    while(~scanf("%d", &n)) {
        init();
        int MaxN = 0;//最大指数
        A[0].x = 1;
        for(register int i = 1; i <= n; i++) {
            int x = read();
            A[x].x = 1;
            MaxN = max(MaxN, x);
        }
        m = read();
        MaxN++;
        for(register int i = 1; i <= m; i++) B[i] = read();
        while(limit <= (MaxN << 1)) limit <<= 1, tot++;
        for(register int i = 0; i < limit; i++) r[i] = (r[i >> 1] >> 1) | ((i & 1) << (tot - 1));
        FFT(A, 1);
        for(register int i = 0; i < limit; i++) A[i] = A[i] * A[i];
        FFT(A, -1);
        for(register int i = 0; i < limit; i++) A[i].x /= limit;
        for(register int i = 1; i <= m; i++) {
            if((int)(A[B[i]].x + 0.5)) ans++;
        }
        write(ans);
    }
    return 0;
}
```

