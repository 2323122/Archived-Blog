---
title: Luogu P1073 最优贸易 解题报告
categories: 解题报告
date: 2019-03-21 16:20:26
tags: 
    - 图论
    - SPFA
top:
---

发一个~~自认为代码比较好看的~~双向SPFA

<!--more-->

# [题目链接](https://www.luogu.org/problemnew/show/P1073)

# 思路

先正图跑一遍SPFA维护每个点对于采购价的最小值。

在数据读入时存反图，以$s$点为源点，再次跑一遍SPFA维护每个点对于卖出价的最小值。

根据题意描述，在存图的时候要注意判断是否为无向边。

要注意本题是点权而不是边权。

# 代码实现

```c++
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<queue>
using std::queue;
using std::min;
using std::max;

const int N = 1e5+6;
const int M = 5e5+6;

int head1[N], next1[M], ver1[M];
int head2[N], next2[M], ver2[M];
int cnt1 = 0, cnt2 = 0;

int a[N];
int d1[N], v1[N], d2[N], v2[N];

int n, m;

inline void add1(int x, int y) {
    ver1[++cnt1] = y, next1[cnt1] = head1[x], head1[x] = cnt1;
}

inline void add2(int x, int y) {
    ver2[++cnt2] = y, next2[cnt2] = head2[x], head2[x] = cnt2;
}

//最小值 
inline void spfa1() {
    queue<int> q1;
    memset(v1,0,sizeof(v1));
    memset(d1,0x3f,sizeof(d1));
    v1[1] = 1, d1[1] = a[1];
    q1.push(1);
    while(!q1.empty()) {
        int x = q1.front();
        q1.pop();
        v1[x] = 0;
        for(register int i = head1[x] ; i ; i = next1[i]) {
            int y = ver1[i];
            if(d1[y] > min(d1[x], a[y])) {
                d1[y] = min(d1[x], a[y]);
                if(!v1[y]) q1.push(y), v1[y] = 1;
            }
        }
    }
}

//最大值 
inline void spfa2() {
    queue<int> q2;
    memset(v2,0,sizeof(v2));
    memset(d2,0,sizeof(d2));
    v2[n] = 1, d2[n] = a[n];
    q2.push(n);
    while(!q2.empty()) {
        int x = q2.front();
        q2.pop();
        v2[x] = 0;
        for(register int i = head2[x] ; i ; i = next2[i]) {
            int y = ver2[i];
            if(d2[y] < max(d2[x], a[y])) {
                d2[y] = max(d2[x], a[y]);
                if(!v2[y]) q2.push(y), v2[y] = 1;
            }
        }
    }
}


int main() {
    scanf("%d%d",&n,&m);
    for(register int i = 1 ; i <= n ; i++) 
        scanf("%d",&a[i]);
    for(register int i = 1 ; i <= m ; i++) {
        int x, y, opt;
        scanf("%d%d%d",&x,&y,&opt);
        if(opt == 1) {
            add1(x,y);
            add2(y,x);
        }
        if(opt == 2) {
            add1(x,y);
            add1(y,x);
            add2(x,y);
            add2(y,x);
        }
    }
    spfa1();
    spfa2();
    int ans = 0;
    for(register int i = 1; i <= n; i++) {
        ans = max(ans,d2[i] - d1[i]);
    }
    printf("%d",ans);
    return 0;
}
```

