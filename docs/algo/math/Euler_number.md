---
comments: true
---

# 欧拉数基础知识

## 前言

模拟赛考到了就稍微了解了一下。不得不说 H 某的集训还是很有好处的，给某谷不少几乎没人做的冷门题目新增了不少题解。

## 欧拉数的定义

欧拉数的具体定义要用到一些高级知识，我 ~~不太了解~~ 完全不知道。但是可以简单地理解为排列中升高个数为 $k$ 的 $n$ 阶排列数，其中“升高”表示 $p_i<p_{i+1}$。用数学语言说，记欧拉数为 $\left\langle\begin{matrix}n\\\\k\end{matrix}\right\rangle$，那么假设 $P$ 是一个排列，$\left\langle\begin{matrix}n\\\\k\end{matrix}\right\rangle$ 即为满足 $\sum_{i=1}^{n-1}[p_i<p_{i+1}]=k$ 的排列数。

## 如何求欧拉数

首先显而易见有个 $O(n^2)$ 的递推，考虑从小到大在序列中插入一个数，那么 $\left\langle\begin{matrix}n\\\\k\end{matrix}\right\rangle$ 显然只会从 $\left\langle\begin{matrix}n-1\\\\k-1\end{matrix}\right\rangle$ 和 $\left\langle\begin{matrix}n-1\\\\k\end{matrix}\right\rangle$ 转移过来（因为最多增加一个升高），容易发现假如 $n$ 插入在原来的升高中间或序列末尾就不会使升高增加，反之就会增加，易得一个最基础的递推式子：

$$\left\langle\begin{matrix}n\\\\k\end{matrix}\right\rangle=(n-k)\left\langle\begin{matrix}n-1\\\\k-1\end{matrix}\right\rangle+(k+1)\left\langle\begin{matrix}n-1\\\\k\end{matrix}\right\rangle$$

就能 $O(n^2)$ 做了。

考虑如何得到更好的式子，我们可以先手推几组小数据。

首先显然 $\left\langle\begin{matrix}n\\\\0\end{matrix}\right\rangle=1$。

对于 $\left\langle\begin{matrix}n\\\\1\end{matrix}\right\rangle$，我们考虑枚举上升在哪个位置，然后计算左右两边边数的集合有多少种取法，易得如下式子：

$$\sum_{i=1}^{n-1}\dbinom{n}{i}$$

但是这个式子是有问题的，假如上升在 $(i,i+1)$ 之间，左边又恰好是 $[n-i+1,n]$，那么整个序列就没有上升了，其它情况显然有上升，由于这种情况会出现 $n-1$ 次（显然），那么实际上答案就是：

$$\left\langle\begin{matrix}n\\\\1\end{matrix}\right\rangle=\left(\sum_{i=1}^{n-1}\dbinom{n}{i}\right)-(n-1)$$

而左边的东西加上 $\binom{n}{0},\binom{n}{n}$ 就是一个二项式定理的形式，故进行一些炫酷的转化后可以化成这样：

$$\left\langle\begin{matrix}n\\\\1\end{matrix}\right\rangle=2^n-n-1$$

然后考虑 $\left\langle\begin{matrix}n\\\\2\end{matrix}\right\rangle$ 怎么求，同样枚举两个升高的位置：

$$\sum_{i=1}^{n-2}\sum_{j=i+1}^{n-1}\dbinom{n}{i}\dbinom{n-i}{j-i}$$

考虑先把这部分化开再去掉不合法的。用类似二项式定理的思路，假设 $(a+b+c)^n$ 某一项除去系数是 $a^xb^yc^{n-x-y}$，那么考虑从 $n$ 个 $(a+b+c)$ 中选 $x$ 个贡献出 $a$，有 $\binom{n}{x}$ 种选法，再选 $b$，有 $\binom{n-x}{y}$ 种选法，然后剩下的没得选，故 $a^xb^yc^{n-x-y}$ 的系数为 $\binom{n}{x}\binom{n-x}{y}$。

回到这道题，上面的式子可以简单化一下得到：

$$\sum_{i=1}^{n-2}\sum_{j=1}^{n-i-1}\dbinom{n}{i}\dbinom{n-i}{j}$$

那么分别添上 $i=0,j=0,n-i+j=0,i=j=0,i=(n-i+j)=0,j=(n-i+j)=0$ 的项，得到：

$$3^n-\left(\dbinom{n}{0}\sum_{j=1}^{n-1}\dbinom{n}{j}\right)-\left(\sum_{i=1}^{n-1}\dbinom{n}{i}\dbinom{n-i}{0}\right)-\left(\sum_{i=1}^{n-1}\dbinom{n}{i}\dbinom{n-i}{n-i}\right)\\\\-\dbinom{n}{0}\dbinom{n}{0}-\dbinom{n}{0}\dbinom{n}{n}-\dbinom{n}{n}\dbinom{n}{0}$$

然后显而易见，上面的式子有几个等于 $1$ 的可以去掉：


$$3^n-\sum_{j=1}^{n-1}\dbinom{n}{j}-\sum_{i=1}^{n-1}\dbinom{n}{i}-\sum_{i=1}^{n-1}\dbinom{n}{i}-3$$

然后后面这三坨用二项式定理搞定：

$$3^n-2^n\times 3+3$$

然后减掉不合法的，首先 $i$ 前面恰好是前 $i$ 大的排列有这么多个：

$$\sum_{i=1}^{n-2}\sum_{j=i+1}^{n-1}\dbinom{n-i}{j-i}$$

这是个杨辉三角前 $n$ 行求和去掉两边边缘的 $1$，大概就是等比数列求和再处理一下，可以得到以下式子：

$$(2^n-n-2)(\dfrac{n^2-n}{2})$$

然后去掉 $i$ 前面的，$j$ 前面全都大于后面的方案数：

$$
\begin{aligned}
 &\sum_{i=1}^{n-2}\dbinom{n}{i}\sum_{j=i+1}^{n-1}1\\\\
=&(2^n-n-2)(\dfrac{(1+n-1)(n-1)}{2})\\\\
=&(2^n-n-2)(\dfrac{n^2-n}{2})
\end{aligned}
$$

~~然后你会发现两个式子化出来长一个样。~~

但是这两种情况有重合，即整个序列是个下降序列的情况，这样的情况有 $\binom{n-1}{2}$ 种。

让我们把所有元素结合在一起：

$$
\begin{aligned}
 &3^n-2^n\times 3+3-2(2^n-n-2)(\dfrac{n^2-n}{2})+\dbinom{n-1}{2}\\\\
=&3^n-2^n\times 3+3-2^n(n^2-n)+(n^3-n^2)+(2n^2-2n)+\dbinom{n-1}{2}\\\\
=&
\end{aligned}
$$