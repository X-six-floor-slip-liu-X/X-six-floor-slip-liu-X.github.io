---
comments: true
---

# 生成函数入门

怎么有人多项式都没搞懂就学 GF 了啊。

[参考资料1](https://www.luogu.com/article/oy8l7j3n) [参考资料2](https://xxeray.gitlab.io/post/generating-function-lecture-notes/)

## 符号与约定

用 $\langle a_i\rangle$ 表示一个数列（不知道为什么很多研究生成函数的 blog 都用尖括号 `\langle \rangle` 表示数列）。

数列 $\xrightarrow{OGF}$ 函数 `\xrightarrow{OGF}` 表示这个函数是这个数列的普通生成函数，$EGF$ 同理。

$F^{\prime}(x)$（`F^{\prime}(x)`）表示 $F(x)$ 的导函数，本文中所有“导数”均指导函数。并且 $F^{(i)}$ 表示 $F$ 的 $i$ 阶导。

## 普通生成函数（OGF）

数列 $\langle a_i\rangle$ 的普通生成函数（ordinary generating function，OGF）定义为：

$$
F(x)=\sum_{i\ge 0}a_ix^{i}
$$

比如 $\langle1,1,1,\dots \rangle\xrightarrow{OGF} F(x)=\sum_{i\ge 0}x^i$。

（个人理解）生成函数是将无穷数列映射到函数（多项式）上的一种工具，通常用于研究数列的规律或通项公式。通常有普通生成函数和指数生成函数两种。

实际上生成函数的映射对象并不是一个多项式，并且这个 $x$ 也不是未知数。这个表示方法其实叫“形式幂级数”，$x$ 叫“代数对象”（虽然未知数也是一种代数对象吧）。“幂级数”是一个非常广义的概念，只要代数对象 $x$ 存在乘法，就能用 $x^k$ 表示其幂次，进而得到幂级数。在生成函数中，$x^k$ 其实是一个对数列下标的“标记”。关注它的“值”是完全没有意义的，我们只关注原数列 $\langle a_i\rangle$，而非 $F(x)$ 的值，“函数值”在生成函数的研究中也是完全没有意义的。因为它的形式和多项式很相近，所以我们通常把它当做多项式处理。

### 封闭形式

在运用生成函数的过程中，我们不会一直使用形式幂级数的形式，而会适时地转化为封闭形式以更好地化简。

对于 $\langle 1,1,1,\dots \rangle$ 的生成函数 $F(x)=\sum_{i\ge 0}x^i$，容易发现：

$$
xF(x)+1=F(x)
$$

于是能得到：

$$
F(x)=\frac{1}{1-x}
$$

这就是 $F(x)$ 的封闭形式。

一般来说，若式子中含有复杂的（无穷或不定）求和，则不便分析。像这样相对更简洁的形式则被称为封闭形式。

如果把它当成多项式，容易发现，当 $|x|\ge 1$ 时 $F(x)$ 显然不收敛。但是“收敛”的定义基于不等式，它是一个关于**值**的不等式性质。而研究生成函数代数对象的“值”是没有意义的。所以研究生成函数时通常忽略收敛性问题，可以认为任何一个含 $x$ 多项式的正无穷次幂都是 $1$（比如上面的式子也能通过带等比数列求和公式把 $x^n$ 这一项换成 $1$ 得到）。

### 运算规律

对于 $\langle a_i \rangle\xrightarrow{OGF}F(x),\langle b_i\rangle\xrightarrow{OGF}G(x)$，有：

$$
F(x)\pm G(x)=\sum_{i\ge 0}(a_i\pm b_i)x^i
$$

则 $F(x)\pm G(x)$ 是 $\langle a_i\pm b_i \rangle$ 的普通生成函数

$$
F(x)\times G(x)=\sum_{i\ge 0}x_i\sum_{j=0}^ia_jb_{i-j}
$$

则 $F(x)G(x)$ 是 $\langle \sum_{j=0}^ia_jb_{i-j} \rangle$ 的普通生成函数。容易发现这是一个卷积的形式。

另外，很多时候我们不好推生成函数的封闭形式，但是我们可以用函数的一些平移等操作通过 $\langle 1,1,1,\dots \rangle$ 的封闭形式推出其余生成函数的封闭形式。

1. 平移（指的是序列的平移而非函数的平移，下同）

$\langle a_0,a_1,a_2,\dots \rangle\to \langle \underbrace{0,0,\dots,0}_c,a_0,a_1,a_2,\dots\rangle$

$F(x)\to x^cF(x)$

2. 拉伸

$\langle a_0,a_1,a_2,\dots \rangle\to \langle a_0,\underbrace{0,0,\dots,0}_c,a_1,\underbrace{0,0,\dots,0}_c,a_2,\dots\rangle$

$F(x)\to F(x^c)$

3. 数乘

$\langle a_0,a_1,a_2,\dots \rangle\to \langle ka_0,ka_1,ka_2,\dots\rangle$

$F(x)\to kF(x)$

4. 乘首项为 $1$ 的等比数列

$\langle a_0,a_1,a_2,\dots \rangle\to \langle c^0a_0,c^1a_1,c^2a_2,\dots\rangle$

$F(x)\to F(cx)$

5. 求导

$\langle a_0,a_1,a_2,\dots \rangle\to \langle a_1,2a_2,3a_3,\dots\rangle$

$F(x)\to F^{\prime}(x)$

6. $n$ 阶导

$\langle a_0,a_1,a_2,\dots \rangle\to \langle n^{\underline n}a_n,(n+1)^{\underline n}a_{n+1},(n+2)^{\underline n}a_{n+2},\dots\rangle$

$F(x)\to F^{(n)}(x)$

上面这些运算规律通常的用法是将生成函数化为 $\sum_{i}\frac{a_i}{1+b_ix^{c_i}}$ 的形式，就能用 $\langle1,1,1,\dots \rangle$ 的生成函数得到对应的序列了。这能在后文看到例子。

但是某些时候你不能（或者很难）将生成函数化成上面的多项式形式，这时我们还有：

- $\sum_{i=0}x^i\binom{n}{i}=(x+1)^n$，直接带二项式定理，非常厉害的是这个 $n$ 可以取任意实数，这时就需要带广义二项式定理。
- $\sum_{i=0}(-x)^i\binom{n}{i}=(1-x)^n$，上面的将 $x$ 取反。
- $\sum_{i=0}x^i\binom{n+i}{i}=\frac{1}{(1-x)^{n+1}}$，需要广义二项式定理。

### 例题

1. 求斐波那契数列的通项公式。

根据定义：

$$
f_i=\begin{cases}
0&,i=0\\\\
1&,i=1\\\\
f_{i-1}+f_{i-2}&,i\ge 2
\end{cases}
$$

设 $f$ 的生成函数为 $F(x)$，有：

$$
\begin{aligned}
F(x)&=\sum_{i\ge 0}x^if_i\\\\
    &=x+\sum_{i\ge 2}x^i(f_{i-1}+f_{i-2})\\\\
    &=x+\left(x\sum_{i\ge 1}x^if_i\right)+\left(x^2\sum_{i\ge 0}x^if_i\right)
\end{aligned}
$$

然后因为 $f_0=0$，所以左边这个可以从 $0$ 枚举。然后容易发现两个 $\sum$ 都变成了 $F(x)$：

$$
\begin{aligned}
F(x)&=x+xF(x)+x^2F(x)\\\\
(1-x-x^2)F(x)&=x\\\\
F(x)&=\frac{x}{1-x-x^2}\\\\
\end{aligned}
$$

但是这玩意显然不太有用，但是我们可以考虑用一些神秘方法把封闭形式展开为幂级数形式，最常用的办法化为多个 $\frac{a}{1-bx^{c}}$ 相加的形式。

那么解一元二次方程暴力对 $1-x-x^2$ 因式分解，得到 $1-x-x^2=(x-\frac{-\sqrt{5}-1}{2})(x-\frac{\sqrt{5}-1}{2})$，为了使公式不太臃肿，下文暂用 $x_1,x_2$ 表示这两个数。

显然 $F(x)$ 能被分成 $\frac{A}{x-x_1}+\frac{B}{x-x_2}$，那么使用待定系数法，易得：

$$
\begin{aligned}
&\frac{A}{x-x_1}+\frac{B}{x-x_2}=\frac{x}{1-x-x^2}\\\\
\Leftrightarrow&A(x-x_2)+B(x-x_1)=x\\\\
\Leftrightarrow&\begin{cases}
Ax+Bx=x\\\\
Ax_2+Bx_1=0
\end{cases}
\end{aligned}
$$

解得：

$$
\begin{cases}
A=\frac{5-\sqrt{5}}{10}\\\\
B=\frac{5+\sqrt{5}}{10}
\end{cases}
$$

则：

$$
\begin{aligned}
F(x)&=\frac{A}{x-x_1}+\frac{B}{x-x_2}\\\\
    &=\frac{\frac{A}{x_1}}{\frac{x}{x_1}-1}+\frac{\frac{A}{x_2}}{\frac{x}{x_2}-1}\\\\
    &=-\left(\frac{\frac{A}{x_1}}{1-\frac{x}{x_1}}+\frac{\frac{A}{x_2}}{1-\frac{x}{x_2}}\right)
\end{aligned}
$$

这样就化成了我们非常喜欢的形式，于是有：

$$
\begin{aligned}
F(x)&=-\left(\frac{A}{x_1}\sum_{i\ge 0}(\frac{x}{x_1})^i+\frac{B}{x_2}\sum_{i\ge 0}(\frac{x}{x_2})^i\right)
\end{aligned}
$$

显然左边这玩意是 $\langle\frac{A}{x_1^1},\frac{A}{x_1^2},\frac{A}{x_1^3},\dots\rangle$ 的生成函数，右边这玩意是 $\langle\frac{B}{x_2^1},\frac{B}{x_2^2},\frac{B}{x_2^3},\dots\rangle$ 的生成函数，于是：

$$
\begin{aligned}
f_i&=-(\frac{A}{x_1^{i+1}}+\frac{B}{x_2^{i+1}})\\\\
   &=\frac{\sqrt{5}}{5}\left(\left(\frac{1+\sqrt{5}}{2}\right)^n-\left(\frac{1-\sqrt{5}}{2}\right)^n\right)
\end{aligned}
$$

这个就是大名鼎鼎的斐波那契数列通项公式。

2. $a_n = 8a_{n-1} + 10^{n - 1},a_0=1$，求 $a$ 的通项公式。

设 $a$ 的生成函数为 $F(x)$，根据定义：

$$
\begin{aligned}
F(x)&=\sum_{i\ge 0}x^ia_i\\\\
    &=a_0+\sum_{i\ge 1}x^i(8a_{i-1}+10^{i-1})\\\\
    &=1+\left(8x\sum_{i\ge 0}x^ia_i\right)+\left(x\sum_{i\ge 0}x^i10^{i}\right)
\end{aligned}
$$

容易发现 $\sum_{i\ge 0}x^i10^{i}$ 是 $\langle10^0,10^1,\dots \rangle$ 的生成函数，那么它的封闭形式就是 $\frac{1}{1-10x}$。而 $\sum_{i\ge 0}x^ia_i$ 就是 $a$ 的生成函数。

于是：

$$
\begin{aligned}
F(x)&=1+8xF(x)+\frac{x}{1-10x}\\\\
(1-8x)F(x)&=\frac{1-9x}{1-10x}\\\\
F(x)&=\frac{1-9x}{(1-8x)(1-10x)}
\end{aligned}
$$

使用待定系数法，设 $\frac{1-9x}{(1-8x)(1-10x)}=\frac{A}{1-8x}+\frac{B}{1-10x}$。解得 $A=\frac{1}{2},B=\frac{1}{2}$。

根据上面的运算法则，容易发现 $\frac{\frac{1}{2}}{1-8x}$ 是 $\langle\frac{1}{2}8^0,\frac{1}{2}8^1,\frac{1}{2}8^2,\dots \rangle$ 的生成函数，$\frac{\frac{1}{2}}{1-10x}$ 是 $\langle\frac{1}{2}10^0,\frac{1}{2}10^1,\frac{1}{2}10^2,\dots \rangle$ 的生成函数，于是 $a_i=\frac{8^i+10^i}{2}$。

3. 求 Catalan 数的通项公式。

给出递推式：

$$
h_n=\begin{cases}
\sum_{i=0}^{n-1}h_ih_{n-i-1} &,n\ge 1\\\\
1 &,n=0
\end{cases}
$$

设 $F(x)$ 为 $h_n$ 的生成函数。

容易发现这个递推式长得形似卷积，考虑化成卷积的样子：

$$
\begin{aligned}
F(x)&=1+\sum_{i\ge 1}x^i\sum_{j=0}^{i-1}h_jh_{i-j-1}\\\\
    &=1+\sum_{i\ge 1}x\sum_{j=0}^{i-1}h_jx^jh_{i-j-1}x^{i-j-1}\\\\
    &=1+x\sum_{j\ge 0}h_jx^j\sum_{k\ge 0}h_{k}x^{k}\\\\
    &=1+xF(x)^2
\end{aligned}
$$

就得到了一个关于 $F(x)$ 的一元二次方程，解得：

$$F(x)=\frac{1\pm \sqrt{1-4x}}{2x}$$

那么应该取哪个根呢？容易发现 $\lim\limits_{x\to 0}F(x)=h_0=1$ 才对，那么我们带进去试一下：

$\lim\limits_{x\to 0}\frac{1+\sqrt{1-4x}}{2x}=\infty$，不对劲。

$\lim\limits_{x\to 0}\frac{1-\sqrt{1-4x}}{2x}=1$，上下均为无穷小。

关于多个解的取舍，不需要精细判别，如果数列存在则必然有且仅有一根正确。一般情况下，我们只需把显然错误的解舍去即可。

然后考虑把封闭形式展开，先来展开 $\sqrt{1-4x}$：

$$
\begin{aligned}
(1-4x)^{\frac{1}{2}}&=\sum_{n\ge 0}\binom{\frac{1}{2}}{n}(-4x)^n\\\\
&=1+\sum_{n\ge 1}\frac{(\frac{1}{2})^{\underline{n}}}{n!}(-4x)^n
\end{aligned}
$$

注意到：

$$
\begin{aligned}
\left(\frac{1}{2}\right)^{\underline{n}}&=\frac{1}{2}\times \frac{-1}{2}\times \frac{-3}{2}\times \cdots \times \frac{-(2n-3)}{2}\\\\
&=\frac{(-1)^{n-1}(2n-3)!!}{2^n}\\\\
&=\frac{(-1)^{n-1}(2n-2)!}{2^n(2n-2)!!}\\\\
&=\frac{(-1)^{n-1}(2n-2)!}{2^{2n-1}(n-1)!}
\end{aligned}
$$

于是带回上式：

$$
\begin{aligned}
&=1+\sum_{n\ge 1}\frac{(-1)^{n-1}(2n-2)!}{n!(n-1)!2^{2n-1}}(-4x)^n\\\\
&=1-\sum_{n\ge 1}\frac{(2n-2)!}{(n-1)!n!}\frac{4^n}{2^{2n-1}}x^n\\\\
&=1-\sum_{n\ge 1}\frac{(2n-2)!}{(n-1)!n!}2x^n\\\\
&=1-\sum_{n\ge 1}\dbinom{2n-1}{n}\frac{1}{2n-1}2x^n\\\\
\end{aligned}
$$

于是：

$$
\begin{aligned}
F(x)&=\frac{1+\sqrt{1-4x}}{2x}\\\\
    &=\frac{1}{2x}\left(1-1+\sum_{n\ge 1}\dbinom{2n-1}{n}\frac{1}{2n-1}2x^n\right)\\\\
    &=\sum_{n\ge 1}\dbinom{2n-1}{n}\frac{1}{2n-1}x^{n-1}\\\\
    &=\sum_{n\ge 0}\dbinom{2n+1}{n+1}\frac{1}{2n+1}x^n\\\\
    &=\sum_{n\ge 0}\frac{(2n+1)!}{(n+1)!n!}\frac{1}{2n+1}x^n\\\\
    &=\sum_{n\ge 0}\frac{(2n)!}{n!n!}\frac{1}{n+1}x^n\\\\
    &=\sum_{n\ge 0}\dbinom{2n}{n}\frac{1}{n+1}x^n\\\\
\end{aligned}
$$

就得到了 Catalan 数的通项公式。

## 指数生成函数（EGF）

一个数列 $\langle a_i \rangle$ 的指数生成函数（exponential generating function，EGF）定义为：

$$
F(x)=\sum_{i\ge 0}a_i\frac{x^i}{i!}
$$

容易发现就是 OGF 每一项额外除以 $i!$。

### 运算规律

加法显然与 OGF 一样，不展开。

考虑乘法，设 $\langle a_0,a_1,a_2,\dots\rangle\xrightarrow{EGF}F(x),\langle b_0,b_1,b_2,\dots\rangle\xrightarrow{EGF}G(x)$，设 $F\times G$ 对应的数列为 $\langle c_0,c_1,c_2,\dots\rangle$，有：

$$
\begin{aligned}
\frac{c_n}{n!}&=\sum_{i+j=n}\frac{a_i}{i!}\frac{b_j}{j!}\\\\
c_n&=\sum_{i+j=n}\frac{n!}{i!j!}a_ib_j\\\\
c_n&=\sum_{i+j=n}\dbinom{n}{i}a_ib_j
\end{aligned}
$$

这个貌似叫“二项加法卷积”。

### 麦克劳林级数

其实是泰勒展开在 $0$ 处的形式，因为搞不懂泰勒展开就先来努力搞这个了。

对于一个**多项式**（注意这里是多项式）

容易发现 $F(x)=\sum_{i\ge 0}\frac{F^{(i)}(0)}{i!}x^i$。

考虑用系数数列表示多项式，$n$ 阶导就是 $\langle  n^{\underline n}a_n,(n+1)^{\underline n}a_{n+1},(n+2)^{\underline n}a_{n+2},\dots\rangle$。容易发现常数项除以 $n!$ 就能得到 $n$ 次项系数。

### 常用 EGF

- $\langle 1,1,1,\dots\rangle\xrightarrow{EGF}e^x$

考虑对 $e^x$ 求麦克劳林级数。因为 $e^x$ 怎么导都是它自己，所以在 $0$ 处均取 $1$。

这个也是 EGF 名字的由来。

- $\langle 1,-1,1,-1,1,-1\dots\rangle\xrightarrow{EGF}e^{-x}$

用 $1$ 变换过来，在 $x$ 的奇数次幂取反。

- $\langle 1,0,1,0,1,0\dots\rangle\xrightarrow{EGF}\frac{e^x+e^{-x}}{2}$

上面两个加起来除以二。

另外上面两个相减除以二可以得到奇数为 $1$ 偶数为 $0$ 的序列。

- $\langle c^0,c^1,c^2,\dots\rangle\xrightarrow{EGF}e^{cx}$

还是用第一个变换过来，相当于 $F(x)\to F(cx)$。

- $\langle 1,a,a^{\underline{2}},a^{\underline{3}},a^{\underline{4}},\dots\rangle\xrightarrow{EGF}(1+x)^a$

广义二项式定理，$a$ 可以取实数。

### 组合意义

若数列的第 $i$ 项表示将 $i$ 个物品组成某种结构的方案数，容易发现 OGF 与 EGF 乘法的组合意义：

- OGF：`{AAAA}+{BBB}->{AAAABBB}`，并不关心集合内元素的相对顺序，相当于**集合的合并**。
- EGF：`{AAA}+{BBB}->{AAABBB},{AABABB},{AABBAB},...`，关注集合内元素的相对顺序，相当于**序列的归并**，比如这个例子就有 $\binom{3+3}{3}$ 种。

并且我们也能发现多项式 exp 的组合意义：

- $\exp F(x)=\sum_{i\ge 0}\frac{F(x)^i}{i!}$。

容易发现，如果把 $F(x)$ 视作“单个结构”的 EGF，那么 $\exp F(x)$ 就描述了若干个结构组成的有标号集合（结构内元素有标号，即“元素”之间有序）。相当于先枚举每个结构的大小，然后多次卷积将其归并起来，由于“结构”之间无序，所以要除以 $i!$。

这个东西非常有用啊。比如说若有标号树的 EGF 为 $F(x)$，那么有标号森林的 EGF 就是 $\exp F(x)$。

并且由于 $\ln$ 是 $\exp$ 的逆运算也能得到 $\ln$ 的组合意义。

### 例题

- 染色问题

将一条长度为 $n$ 的纸条涂成红绿蓝三种颜色，每段颜色必须覆盖整数长度，使得红色和蓝色的个数均为偶数，求方案数（的通项公式）。

考虑每种颜色涂了多少个，然后“归并”成一条纸条。这恰好是 EGF 卷积的形式。

对于绿色，涂任意长度都是合法的，于是有 $\langle 1,1,1,\dots \rangle\xrightarrow{EGF}e^x$。

对于红色和蓝色，只有奇数合法，于是有 $\langle 1,0,1,0,1,\dots \rangle\xrightarrow{EGF}\frac{e^x+e^{-x}}{2}$。

则总方案数 EGF 的封闭形式为：

$$
\left(\frac{e^x+e^{-x}}{2}\right)^2e^x=\frac{e^{3x}+2e^x+e^{-1}}{4}
$$

于是能得到第 $n$ 项系数为 $\frac{3^n+2+(-1)^n}{4}$。

- 有标号二分图计数

不要求连通。

一个显然的想法是枚举左部点集合（不妨令大小为 $a$），然后方案数就是 $2^{a(n-a)}$。

但是注意到一个不连通二分图有不止一种划分左右部点的方法，而且左右部点交换也会被算一次，于是会算重。那么容易想到先求出它的 EGF，$\ln$ 一下就能求出连通二分图个数的 EGF，再除以二就能去掉交换左右部点的方案，最后 $\exp$ 回去就能得到答案了。

发现若设 $b_n=\sum_a \binom{n}{a}2^{a(n-a)},\langle b\rangle\xrightarrow{EGF} F(x)$，那么答案就是 $\exp(\frac{1}{2}\ln F(x))=\sqrt{F(x)}$。