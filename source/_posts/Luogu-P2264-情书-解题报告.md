---
title: Luogu P2264 情书 解题报告
categories: 解题报告
toc: true
date: 2019-09-05 19:11:26
tags:
	- 字符串处理
	- 模拟
top:
thumbnail: https://i.loli.net/2019/09/05/KPXHlwonsWR6S84.jpg
---

模拟太好玩了。

# 题目链接

[P2264 情书](https://www.luogu.org/problem/P2264)

# 题意简述

给你 $n$ 个单词，让你从一篇文章中找出这些单词总出现次数，每句话中一个单词出现多次只贡献一次次数。不区分大小写。

<!--more-->

# 解题思想

直接模拟即可。

说一下我写的过程中遇到的一些 ~~sb~~ 问题。

首先是要注意非句点和字母的垃圾字符，这些字符会对一堆字符是否构成单词造成影响，需要判一下。

给定的单词也要转换大小写，建议都换为小写。

一个句点不一定代表全文结束，此时需要初始化维护单词是否出现过的数组。

顺带一提，当代码中有 `while(scanf("%xxx", xxx) != EOF)`  语句时，使用控制台测样例的方法是：
在输入结束后，按下 `Ctrl + z` ，**然后再按回车**。

# 参考代码

```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <string>

using namespace std;

const int N = 110;

string a[N], now;
char ch;
bool v[N];

int n, ans = 0;

int main() {
    scanf("%d", &n);
    for(register int i = 1; i <= n; i++) {
        cin >> a[i];
        int len = a[i].length();
        for(register int j = 0; j < len; j++) {
            if(a[i][j] >= 'A' && a[i][j] <= 'Z') a[i][j] = a[i][j] - 'A' + 'a';
        }
    }
    while(scanf("%c", &ch) != EOF) {
        if(ch >= 'A' && ch <= 'Z') {
            ch = ch - 'A' + 'a';
        }
        if(ch < 'a' || ch > 'z') {
            if(now[0] < 'a' || now[0] > 'z') continue;
            for(register int i = 1; i <= n; i++) {
                if(now == a[i] && !v[i]) {
                    v[i] = 1;
                    ans++;
                    break;
                }
            }
            if(ch == '.') {
                memset(v, 0, sizeof v);
            }
            now.clear();
        }
        else now += ch;
    }
    printf("%d", ans);
    return 0;
}
```

