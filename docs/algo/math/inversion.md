---
comments: true
---

# 线性反演入门

听了一下午啥都没学会，考虑造一下轮子（指写一篇一样的文章）。

本文中“反演”均指线性反演。

## 什么是反演

反演是一个定义在线性变换上的关系。

称 $T$ 和 $T'$ 两个矩阵为反演关系当且仅当 $x\times  T\times T'=x$。

容易发现 $T$ 和 $T'$ 互逆，根据矩阵乘法的结合律易证。

举个栗子，设 $x$ 是数列，那么前缀和和差分互为反演关系，因为从 $x$ 作前缀和得到 $x'$ 再做差分就回来了。

考虑前缀和的矩阵（此处考虑把 $x$ 当成一个行向量右乘转移矩阵，我们通常把这样描述线性关系的矩阵叫**关系矩阵**）是 $T_{i,j}=[j\ge i]$，差分的矩阵是 $T_{i,j}'=[j=i]-[j=i+1]$，容易发现两矩阵相乘就能得到单位矩阵 $I$。

那么线性代数比较强的同学就能用矩阵的知识来分析反演了。

并且容易发现 OI 范围内讨论的反演其实就是一堆容斥套路，通常用来解决计数问题。比如有时候要求一个函数 $F(n)$，但它不好求，而存在一个 $G=F\times T$ 是好求的，并且存在 $T'$ 和 $T$ 为反演关系，那么可以用 $F=G\times T'$ 来求出 $F$。

## 二项式反演

比较基础的反演，后面很多东西都需要用这个证明。关系矩阵是把杨辉三角横过来填到上三角矩阵，然后奇数行全取相反数。

形式化的（此处的 $G(x),F(x)$ 均为数列）：

$$
G(n)=\sum_{i=0}^n(-1)^i\dbinom{n}{i}F(i)
$$

然后用矩阵描述就是：

$$
T_{i,j}=(-1)^i\dbinom{j}{i}
$$

它和自己互为反演关系，考虑把两个关系矩阵乘起来：

$$
\begin{aligned}
 &(T\times T)\_{i,j}\\\\
=&\sum_{k=0}^nT_{i,k}\times T_{k,j}\\\\
=&\sum_{k=j}^i(-1)^k\binom{i}{k}(-1)^j\binom{k}{j}\\\\
=&(-1)^j\sum_{k=j}^i(-1)^k\binom{i}{j}\binom{i-j}{k-j}\\\\
=&(-1)^j\binom{i}{j}\sum_{k=j}^i(-1)^k\binom{i-j}{k-j}\\\\
\end{aligned}
$$

其中第三步能改 $\sum$ 的范围是因为 $[j,i]$ 外面 $\binom{i}{k}\binom{k}{j}=0$，第四步是经典组合意义，$i$ 个里面选 $k$ 个再从 $k$ 个里面选 $j$ 个等价于直接选 $j$ 个然后在剩下的里面选出第一步选了但第二步没选的。

容易发现 $\sum_{k=j}^i(-1)^k\binom{i-j}{k-j}$ 是杨辉三角第 $i-j$ 这一整行中偶数列减去奇数列。根据经典结论，这个东西不等于 $0$ 当且仅当 $i-j=0$，原因是当 $i-j> 0$ 时杨辉三角前一行每个数都能对这一行的奇数之和和偶数之和各产生一次贡献。

于是容易发现 $(T\times T)\_{i,j}=I$，故 $T$ 和自己互为反演关系，于是能得到这样的式子：

$$
G(n)=\sum_{i=0}^n(-1)^i\dbinom{n}{i}F(i)
$$
$$
F(n)=\sum_{i=0}^n(-1)^i\dbinom{n}{i}G(i)
$$

考虑一些简单的变换：

$$
\begin{aligned}
F(n)&=\sum_{i=0}^n(-1)^i\dbinom{n}{i}G(i)\\\\
(-1)^{-n}F(n)&=\sum_{i=0}^n(-1)^{i-n}\binom{n}{i}G(i)\\\\
(-1)^nF(n)&=\sum_{i=0}^n(-1)^{n-i}\binom{n}{i}G(i)
\end{aligned}
$$

其中第三步指数取相反数是因为 $(-1)^x=(-1)^{-x}$，而不是因为两边指数同时取相反数有什么神秘性质。

然后令 $H(n)=(-1)^nF(n)$，易得：

$$
G(n)=\sum_{i=0}^n\binom{n}{i}H(i)
$$

$$
H(n)=\sum_{i=0}^n(-1)^{n-i}\binom{n}{i}G(i)
$$

这个更好看的形式。而且这个结论非常强力，只要式子 $1$ 成立，那么式子 $2$ 成立。

容易发现，可以用 $G(n)$ 表示从 $n$ 个中选任意个组成某种特定结构的方案数，$F(n)$ 表示用恰好 $n$ 个组成特定结构的方案数。通常 $G$ 是比 $F$ 好算的，那么就能通过这个式子用 $G$ 求出 $F$ 了。通常来说，二项式反演是把**恰好**转化为**至多**来解题了。

### 经典问题：错排问题

这里以错排问题为例，即有多少个 $n$ 阶排列 $p_i$，满足 $\forall_{i=1}^na_i\ne i$。

那么可以令 $H(n)$ 表示 $n$ 阶错排，$G(n)$ 表示 $n$ 阶排列。

显然 $G(n)=n!$，且 $G(n)=\sum_{i=0}^n\binom{n}{i}H(i)$（考虑其它地方不动，选 $i$ 个搞成错排）。

于是有：

$$
\begin{aligned}
H(n)&=\sum_{i=0}^n(-1)^{n-i}\binom{n}{i}G(i)\\\\
    &=\sum_{i=0}^n(-1)^{n-i}\frac{n!}{i!(n-i)!}i!\\\\
    &=n!\sum_{i=0}^n\frac{(-1)^{i}}{i!}\\\\
\end{aligned}
$$

其中第三步是直接改成枚举 $n-i$。

这就是经典的错排公式。

## 子集反演

和二项式反演非常类似。

$$
G(A)=\sum_{B\subseteq A}H(B)
$$
$$
H(A)=\sum_{B\subseteq A}(-1)^{|A|-|B|}G(B)
$$

其中 $G(S),H(S)$ 均是定义在集合上的函数，$A,B$ 为集合。

我们把集合离散化到整数上，然后对这个东西定义矩阵 $T,T'$，考虑用 $x\times T\times T'=x$ 来证明。则：

$$
\begin{aligned}
H(A)&=\sum_{B\subseteq A}(-1)^{|A|-|B|}G(B)\\\\
    &=\sum_{B\subseteq A}(-1)^{|A|-|B|}\sum_{C\subseteq B}H(C)\\\\
    &=\sum_{C\subseteq A}H(C)\sum_{C\subseteq B\subseteq A}(-1)^{|A|-|B|}
\end{aligned}
$$

容易发现 $\sum_{C\subseteq B\subseteq A}(-1)^{|A|-|B|}=\sum_{k=|C|}^{|A|}(-1)^{|A|-k}\binom{|A|-|C|}{k-|C|}$，当且仅当 $|A|=|C|$ 时为 $1$，其余时候为 $0$。则 $\sum_{C\subseteq A}H(C)\sum_{C\subseteq B\subseteq A}(-1)^{|A|-|B|}=H(A)$，得证。

然后使用也和二项式反演类似，把**恰为集合** $S$ 转化为 **是集合 $S$ 的子集**来解决问题。

## 斯特林反演

前置知识：[斯特林数](./stirling.md)。

斯特林反演形如以下式子：

$$
G(n)=\sum_{i=0}^n\begin{Bmatrix}n\\\\i\end{Bmatrix}F(i)
$$
$$
F(n)=\sum_{i=0}^n(-1)^{n-i}\begin{bmatrix}n\\\\i\end{bmatrix}G(i)
$$

证明非常神秘：

考虑转移矩阵乘起来长这样：

$$
\begin{aligned}
 &(T\times T')\_{i,j}\\\\
=&\sum_{k=0}^nT_{i,k}\cdot T_{k,j}'\\\\
=&\sum_{k=j}^i(-1)^{k-j}\begin{Bmatrix}i\\\\k\end{Bmatrix}\begin{bmatrix}k\\\\j\end{bmatrix}\\\\
\end{aligned}
$$

我们希望证明 $\sum_{k=j}^i(-1)^{k-j}(-1)^{k-j}\begin{Bmatrix}i\\\\k\end{Bmatrix}\begin{bmatrix}k\\\\j\end{bmatrix}=[i=j]$

根据经典结论，有：

$$
\begin{aligned}
m^{\underline{n}}&=\sum_{i=0}^n(-1)^{n-i}\begin{bmatrix}n\\\\i\end{bmatrix}m^i\\\\
                 &=\sum_{i=0}^n(-1)^{n-i}\begin{bmatrix}n\\\\i\end{bmatrix}\sum_{j=0}^i\begin{Bmatrix}i\\\\j\end{Bmatrix}m^{\underline{j}}\\\\
                 &=\sum_{i=0}^nm^{\underline{i}}\sum_{j=i}^n(-1)^{n-j}\begin{bmatrix}n\\\\j\end{bmatrix}\begin{Bmatrix}j\\\\i\end{Bmatrix}
\end{aligned}
$$

那么显然当 $i\ne n$ 时，$\sum_{j=i}^n(-1)^{n-j}\begin{bmatrix}n\\\\j\end{bmatrix}\begin{Bmatrix}j\\\\i\end{Bmatrix}=0$，否则等号无法成立。

则 $\sum_{k=j}^i(-1)^{k-j}(-1)^{k-j}\begin{Bmatrix}i\\\\k\end{Bmatrix}\begin{bmatrix}k\\\\j\end{bmatrix}=[i=j]$，得证。

然后和二项式反演相同，我们可以移动 $(-1)^{n-i}$ 的位置（取 $F'(n)=(-1)^{-n}F(n)=(-1)^{n}F(n),G'(n)=(-1)^{-n}G(n)=(-1)^nG(n)$）。于是：


$$
G'(n)=\sum_{i=0}^n(-1)^{n-i}\begin{Bmatrix}n\\\\i\end{Bmatrix}F'(i)
$$
$$
F'(n)=\sum_{i=0}^n\begin{bmatrix}n\\\\i\end{bmatrix}G'(i)
$$

常用做法还是用来做计数题。

## 莫比乌斯反演

右转[狄利克雷卷积与莫比乌斯反演入门](./Mobius_inversion.md)。