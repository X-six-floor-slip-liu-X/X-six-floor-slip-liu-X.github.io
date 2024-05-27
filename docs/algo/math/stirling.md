---
comments: true
---

# 斯特林数学习笔记

## 第二类斯特林数

为什么先说第二类斯特林数？ ~~因为 OIwiki 是先讲的第二类~~ 

定义 $\begin{Bmatrix}n\\\\k\end{Bmatrix}$ 表示将 $n$ 个两两不同的元素划分为 $k$ 个互不区分的非空子集的方案数。称 $\begin{Bmatrix}n\\\\k\end{Bmatrix}$ 为**第二类斯特林数**。

### 递推式

有个非常简单的递推式：

$$
\begin{Bmatrix}n\\\\k\end{Bmatrix}=\begin{Bmatrix}n-1\\\\k-1\end{Bmatrix}+k\begin{Bmatrix}n-1\\\\k\end{Bmatrix}
$$

边界是 $\begin{Bmatrix}n\\\\0\end{Bmatrix}=[n=0]$。

考虑新加的第 $n$ 个元素。$\begin{Bmatrix}n-1\\\\k-1\end{Bmatrix}$ 表示将这个元素划分为一个新的集合，后面的则是把这个元素加入之前的一个集合。用加法原理加起来即可。

### 通项公式

可以用[二项式反演](./inversion.md)套路地做。

考虑 $G(i)$ 表示将 $n$ 个元素划分到两两不同的 $i$ 个集合的方案数（可以为空），$F(i)$ 表示将 $n$ 个元素划分到 $i$ 个两两不同的非空集合的方案数。

显然有： $G(i)=i^n,G(i)=\sum_{j=0}^i\binom{i}{j}F(j)$，那么：

$$
\begin{aligned}
F(i)&=\sum_{j=0}^i(-1)^{i-j}\binom{i}{j}G(j)\\\\
    &=\sum_{j=0}^i\frac{(-1)^{i-j}i!j^n}{j!(i-j)!}
\end{aligned}
$$

然后第二类斯特林数是划分成 $k$ 个**互不区分**的集合，所以每种划分会被 $F(k)$ 算 $k!$ 遍，除去即可。

综上：

$$
\begin{Bmatrix}n\\\\k\end{Bmatrix}=\sum_{j=0}^k\frac{(-1)^{k-j}j^n}{j!(k-j)!}
$$

### 同一行第二类斯特林数的计算

留坑待补。

### 同一列第二类斯特林数的计算

留坑待补。

## 第一类斯特林数

定义 $\begin{bmatrix}n\\\\k\end{bmatrix}$ 表示将 $n$ 个两两不同的元素划分为 $k$ 个互不区分的非空轮换的方案数。

一个轮换就是一个首尾相接的环形序列，可以视作一串珠子。比如 $1,2,3,4$ 和 $2,3,4,1$ 是同一个轮换，但 $1,2,3,4$ 和 $4,3,2,1$ 不是同一个轮换。

### 递推式

$$
\begin{bmatrix}n\\\\k\end{bmatrix}=\begin{bmatrix}n-1\\\\k-1\end{bmatrix}+(n-1)\begin{bmatrix}n-1\\\\k\end{bmatrix}
$$

边界是 $\begin{bmatrix}n\\\\0\end{bmatrix}=[n=0]$。

证明和第二类类似，关于系数 $n-1$，因为插到前面每个数都后面是不同的。

### 通项公式

第一类斯特林数没有实用的通项公式。

### 同一行第一类斯特林数的计算

留坑待补。

### 同一列第一类斯特林数的计算

留坑待补。

## 应用

### 上升幂和下降幂转化为普通幂

记上升幂 $x^{\overline{n}}=\prod_{k=0}^{n-1}(x+k)$，则有：

$$
x^{\overline{n}}=\sum_{k=0}^n\begin{bmatrix}n\\\\k\end{bmatrix}x^k
$$

/// details | 证明
    open: True
    type: note

考虑数学归纳法。

首先对于 $n=0$，等号两边都等于 $1$。对于 $n=1$，左右两边都等于 $x$。

考虑 $n>1$，此时有：

$$
\begin{aligned}
x^{\overline{n}}&=(x+n-1)x^{\overline{n-1}}\\\\
                &=(x+n-1)\sum_{k=0}^{n-1}\begin{bmatrix}n-1\\\\k\end{bmatrix}x^k\\\\
                &=\left(x\sum_{k=0}^{n-1}\begin{bmatrix}n-1\\\\k\end{bmatrix}x^k\right)+\left((n-1)\sum_{k=0}^{n-1}\begin{bmatrix}n-1\\\\k\end{bmatrix}x^k\right)\\\\
                &=\left(\sum_{k=1}^{n}\begin{bmatrix}n-1\\\\k-1\end{bmatrix}x^k\right)+\left(\sum_{k=1}^{n}(n-1)\begin{bmatrix}n-1\\\\k\end{bmatrix}x^k\right)\\\\
                &=\sum_{k=1}^n\left(\begin{bmatrix}n-1\\\\k-1\end{bmatrix}+(n-1)\begin{bmatrix}n-1\\\\k\end{bmatrix}\right)x^k\\\\
                &=\sum_{k=1}^n\begin{bmatrix}n\\\\k\end{bmatrix}x^k\\\\
                &=\sum_{k=0}^n\begin{bmatrix}n\\\\k\end{bmatrix}x^k
\end{aligned}
$$

对于其中第四步两个 $\sum$ 下标改变的原因，第一个是将 $x$ 乘进去后改为枚举 $k+1$，第二个是因为 $k=0,k=n$ 时 $\begin{bmatrix}n-1\\\\k\end{bmatrix}$ 均为 $0$。最后一步也同理。

///

记下降幂 $x^{\underline{n}}=\prod_{k=0}^{n-1}(x-k)$，有：

$$
x^{\underline{n}}=\sum_{k=0}^n(-1)^{n-k}\begin{bmatrix}n\\\\k\end{bmatrix}x^k
$$

证明类似，守序邪恶。

![证明](../../img/proof.png)


### 普通幂转化为上升幂和下降幂

普通幂转下降幂：

$$
\begin{aligned}
x^n&=\sum_{k}\begin{Bmatrix}n\\\\k\end{Bmatrix}k!\binom{x}{k}\\\\
   &=\sum_{k}\begin{Bmatrix}n\\\\k\end{Bmatrix}k!\frac{x!}{(x-k)!k!}\\\\
   &=\sum_{k}\begin{Bmatrix}n\\\\k\end{Bmatrix}x^{\underline{k}}
\end{aligned}
$$

第一步考虑组合意义，$n$ 个球放进 $x$ 个互相区分的可空盒子里，那么先从 $x$ 个盒子里面选 $k$ 个，然后转化成 $n$ 个放到 $K$ 个互相区分的非空盒子里。后面的推导都很显然。

更板的推导是使用[斯特林反演](./inversion.md)。

然后普通幂转上升幂：

$$
x^n=\sum_{k}(-1)^{n-k}\begin{Bmatrix}n\\\\k\end{Bmatrix}x^{\overline{k}}
$$

证明还是斯特林反演。