---
title: notes
date: 2019-08-20 18:40:34
---

这里会放一些Yuki记的小知识点和注意事项$\sim$

最近更新的会放在最上面

------

[$\LaTeX$公式大全](https://www.luogu.org/blog/IowaBattleship/latex-gong-shi-tai-quan)

感谢原作者[Iowa_BattleShip](https://www.luogu.org/space/show?uid=60181)

-------------------

1. SA 卡时防 TLE 应该把设定时限开到时限的 $\frac{1}{2}$，这样不会因为第二次 `SA()` 而超时。写法：`while(Time() < LIMIT / 2) SA();`

2. 对数组 `double a[N]` 赋初值不能用 `memset(a, 0x3f, sizeof a)` ，而应该用 `memset(a, 127, sizeof a)`。

3. 分块题用均值不等式算出的块的大小，把 $n$ 的上界带进去，对于所有数据块都开一样大的就行了。

4. 数据结构题读判断符用 `scanf(" %c", &op);` `%c` 前要带空格。

5. `set<Node> s` 中，`Node` 结构体不能重载`==`号，但是可以通过重载`<`来实现重载`>`和`==`。

6. 调试不要直接写输出代码，要 `#ifdef DEBUG` 并添加编译选项 `-D DEBUG`。

7. **写线段树开 `long long` 不要 `#define ll long long`，会跟 `query(o, l, r, ll, rr)` 冲突！！！**

8. $\sum\limits_{d|\gcd(i, j)}\mu(d) = [\gcd(i, j) = 1]$

    证明：

    莫比乌斯函数性质得：$\sum\limits_{d|n}{\mu(d)} = [n = 1]$

    将 $n$ 替换为 $\gcd(i, j)$即得证    

9. $\sum\limits_{i = 1}^{n} \sum\limits_{j = 1}^{m}a_ib_j = \sum\limits_{i = 1} ^ {n} a_i \times \sum\limits_{j = 1}^{m}b_j$

    证明：

    拆开后合并同类项。

10. 数列 $<1,  \dbinom{c}{1},  \dbinom{c + 1}{2},  \dbinom{c + 2}{3}, \cdots>$ 的生成函数为 $G(z) = \frac{1}{(1-z) ^ c}$

    证明：
    从右边考虑：

    $\frac{1}{(1-z) ^ c} = (1 - z) ^ {-c}$

    广义二项式定理展开：
    原式
    $= \sum\limits_{k\geq 0}^{\infty}\dbinom{-c}{k}\times (-z) ^ k$

    $= {\sum\limits_{k\geq 0}^{\infty} \frac{(-c)^{\underline{k}}}{k!}\times (-1) ^ k \times z ^ k} ^ {[1]}$

    其中：
    ${(-c)} ^ {\underline{k}} $

    $= (-c) \times (-c - 1) \times (-c - 2) \times\cdots\times (-c - k + 1)$

    $= (-1) ^ k \times c\times (c + 1) \times (c + 2) \times \cdots \times (c + k - 1)$

    $= (-1) ^ k \times (c + k - 1) ^ {\underline{k}}$

    代回原式：
    原式
    $=\sum\limits_{k \geq 0} ^ {\infty}\frac{(-1)^k\times (c + k - 1) ^{\underline{k}}}{k!} \times (-1) ^ k \times z ^ k$

    $= \sum\limits_{k \geq 0} ^ {\infty}\frac{(-1) ^ {2k} \times (c + k - 1) ^ {\underline{k}}}{k!} \times z^k$

    又因为：
    $(-1)^{2k} = 1$

    所以：
    原式 $=\sum\limits_{k\geq 0}^{\infty}\frac{(c + k - 1) ^ {\underline{k}}}{k!}\times z^k$

    由组合数推广定义：
    原式 $=\sum\limits_{k\ge 0} ^ {\infty} \dbinom{c + k - 1}{k} \times z ^ k$

    是由定义得到的原数列的生成函数。

${}^1a ^ {\underline{k}} = \prod\limits_{i = 1} ^ {k} (a - i + 1)$