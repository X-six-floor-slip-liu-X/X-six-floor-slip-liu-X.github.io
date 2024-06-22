---
comments: true
---

# 多项式基础

[参考资料 1](https://www.luogu.com/article/qdzrwqrd)

[参考资料 2](https://xxeray.gitlab.io/post/generating-function-lecture-notes)

## 符号与约定

所有操作均默认是在模意义 $\pmod{998244353}$ 下进行的，因为 NTT 需要模这个数。

称多项式“项数”为其次数加一（因为还有 $x^0$ 这一项）。

用 $F[i]$ 表示多项式 $F(x)$ 中 $x^i$ 这一项的系数。

默认 $n$ 代表某多项式的项数。

很多时候都用 $n'$ 表示一个和 $n$ 相关的代数对象，但是因为在函数中这个符号会和导数产生歧义，故用 $F_{\ast}(x)$ 表示一个和 $F(x)$ 相关的函数。

有时候会用 $(F\circ G)(x)$ 表示 $F(G(x))$（即多项式的复合，目的是在取导数的时候将 $F'(G(x))$（以 $G(x)$ 为主元）和 $(F\circ G)'(x)$（以 $x$ 为主元）区分开）。

## 多项式四则运算

- 加减法

多项式的加减法定义为：

$$
F(x)\pm G(x)=\sum_{i\ge 0}(F[i]\pm G[i])x^i
$$

就是对应项系数相加减，可以直接 $O(n)$。

- 乘法

多项式乘法的定义为：

$$
F(x)\times G(x)=\sum_{i\ge 0}\sum_{j\ge 0}F[i]G[j]x^{i+j}=\sum_{i\ge 0}x^i\sum_{j=0}^i F[j]G[i-j]
$$

其实就是**加法卷积**。直接暴力是 $O(n^2)$ 的。但是可以用 NTT 优化到 $O(n\log n)$，移步[这里](./FFT_NTT.md)阅读。你在这里看到乘法的时候应该反应过来需要用 NTT 实现。

需要注意的是 $n$ 项多项式乘以 $m$ 项多项式会得到 $n+m$ 项多项式，这点在后面会再次说明。

- 除法

参见后文“多项式求逆”。

- 关于运算的合理性

本文所讨论的多项式均为**形式幂级数**，但限于篇幅，并没有给出其严格定义和应用。

它只是辅助分析的数学工具，所谓多项式中的“未知数”并没有实际意义，只是一个**代数对象**。比如在生成函数中我们用代数对象的幂次来标记数列的下标。

所以我们其实只关注多项式的系数，故可以用**数列加法**和**加法卷积**来定义多项式的运算。

- 界

多数时候，我们只关注多项式的前若干项，所以我们通常给多项式设置了一个次数上界，称为**界**，相当于 $\bmod\; x^n$，表示我们只关注 $x^0$ 项到 $x^{n-1}$ 项。

由于多项式加法不会对代数对象的幂次产生影响，故显然有：

$$
(F(x)\bmod x^n+G(x)\bmod x^n)\bmod x^n=(F(x)+G(x))\bmod x^n
$$

而由于乘法只会从低次项向高次项转移，故也有：

$$
((F(x)\bmod x^n)\times (G(x)\bmod x^n))\bmod x^n=(F(x)\times G(x))\bmod x^n
$$

但是如刚才所说，注意 $(F(x)\bmod x^n)\times (G(x)\bmod x^n)$ 有 $2n-1$ 项，在 NTT 时应该把 $n'$ 设为 $2n-1$ 而非 $n$，不要产生像计数题里面对某个数取模就不用担心爆 `long long` 种惯性思维。

## 多项式进阶运算

### 泰勒展开

设 $F^{(i)}(x)$ 表示 $F(x)$ 的 $i$ 阶导，有：

$$
F(x)=\sum_{i\ge 0}\frac{F^{(i)}(x_0)}{i!}(x-x_0)^i
$$

其中 $x_0$ 是你任意取的一个数。

这个东西可以把一些神秘式子换成多项式，在一些证明中非常有用。

具体推导十分复杂，此处略过。

### 牛顿迭代法

一个研究多项式时非常好用的工具，若要阅读后文请确保你理解了这一段。

对于一个已知函数 $F(x)$，若存在一个与它相关的函数 $H(x)$（此处“相关”指存在 $G(H)=0$，其中 $G$ 是一个定义域为多项式的函数，且 $G$ 的定义式中含有 $F$，具体例子见后文），我们可以用牛顿迭代求出 $G(x)\pmod{x^n}$。

有：

$$
H\equiv H_{\ast}-\frac{G(H_{\ast})}{G'(H_{\ast})}\pmod{x^n}
$$

其中 $H_{\ast}$ 满足 $G(H_{\ast})\equiv 0\pmod{x^{\frac{n}{2}}}$，需要钦定 $n$ 是偶数。

注意这里的 $G'(H_{\ast})$ 是以 $H_{\ast}$ 为主元的，即 $\frac{\operatorname{d} G(H_*(x))}{\operatorname{d} H_*(x)}$。

/// admonition | 证明
    type: quote

根据定义，显然有 $H(x)\equiv H_{\ast}(x)\pmod{x^{\frac{n}{2}}}$。将 $G(H)$ 在 $H_{\ast}(x)$ 处泰勒展开：

$$
G(H)=\sum_{i\ge 0}\frac{G^{(i)}(H_{\ast})}{i!}(H-H_{\ast})^i
$$

注意到 $H$ 与 $H_{\ast}$ 的前 $\frac{n}{2}$ 项相同。于是 $H-H_{\ast}$ 就只有后 $\frac{n}{2}$ 项。那么容易发现当 $c\ge 2$ 时，有 $(H-H_{\ast})^c\equiv 0\pmod{x^n}$。于是上面这个式子就只有前两项有用了，于是有：

$$
\begin{aligned}
G(H)&\equiv G(H_{\ast})+G'(H_{\ast})(H-H_{\ast}) &\pmod{x^n}\\\\
G'(H_{\ast})H&\equiv G(H)-G(H_{\ast})+G'(H_{\ast})H_{\ast} &\pmod{x^n}\\\\
H&\equiv \frac{G(H)-G(H_{\ast})+G'(H_{\ast})H_{\ast}}{G'(H_{\ast})} &\pmod{x^n}\\\\
H&\equiv H_{\ast}+\frac{G(H)-G(H_{\ast})}{G'(H_{\ast})} &\pmod{x^n}\\\\
H&\equiv H_{\ast}+\frac{G(H)-G(H_{\ast})}{G'(H_{\ast})} &\pmod{x^n}\\\\
\end{aligned}
$$

欸！我们注意到 $G(H)\equiv 0\pmod{x^n}$，于是：

$$
\begin{aligned}
H&\equiv H_{\ast}-\frac{G(H_{\ast})}{G'(H_{\ast})} &\pmod{x^n}\\\\
\end{aligned}
$$

///