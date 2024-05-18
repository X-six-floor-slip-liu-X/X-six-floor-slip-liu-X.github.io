---
comments: true
---

# Barrett 取模优化

## 前言

今天模拟赛有人用双 $\log$ 做法加 Barrett 优化卡过正解单 $\log$ 都要卡常的题了，感觉很有意思，学点新东西,记录一下。

另外不知道这个算法怎么分类（因为沾点数学但又是只有写代码时才有用的）索性放到杂项里了。

## Barrett 是什么

Barrett 是用来优化对同一个数的取模的，在固定模数 $P$ 的情况下，Barrett 的速度比 C++ 自带的 `%` 要快得多。

## Barrett 的原理是什么

首先考虑取模写成四则运算是什么，容易发现 $a\bmod P=a-P\left\lfloor\frac{a}{P}\right\rfloor$。也就是说，我们只要优化了算 $\left\lfloor\frac{a}{P}\right\rfloor$ 的效率，就能加速取模。

由于 C++ 默认的除法效率并不高，考虑用估算的方法，使得 $\frac{1}{P}\approx\frac{m}{2^k}$，因为计算机算乘法是很快的，而 $\frac{1}{2^k}$ 可以用右移运算解决。那么此时 $m\approx\frac{2^k}{P}$，不妨设 $m=\left\lfloor\frac{2^k}{P}\right\rfloor$，去掉下取整就是 $m=\frac{2^k}{P}-\lambda$，其中 $0\le \lambda<1$。

现在用这个近似值估算：

$$\left\lfloor\dfrac{a}{P}\right\rfloor\approx\left\lfloor\dfrac{am}{2^k}\right\rfloor=\left\lfloor a\left(\dfrac{\frac{2^k}{P}-\lambda}{2^k}\right)\right\rfloor=\left\lfloor\dfrac{a}{P}-\dfrac{a\lambda}{2^k}\right\rfloor$$

假如 $k$ 足够大，使得 $\dfrac{a\lambda}{2^k}<1$，那么分类讨论一下就能得到以下式子：

$$
\left\lfloor\dfrac{a}{P}-\dfrac{a\lambda}{2^k}\right\rfloor=\begin{cases}
\left\lfloor\dfrac{a}{P}\right\rfloor, &\dfrac{a}{P}-\dfrac{a\lambda}{2^k}\ge\left\lfloor\dfrac{a}{P}\right\rfloor\\\\
\left\lfloor\dfrac{a}{P}\right\rfloor-1,&\dfrac{a}{P}-\dfrac{a\lambda}{2^k}<\left\lfloor\dfrac{a}{P}\right\rfloor
\end{cases}
$$

然后塞进原式里：

$$
a-P\left\lfloor\dfrac{a}{P}-\dfrac{a\lambda}{2^k}\right\rfloor=\begin{cases}
a-P\left\lfloor\dfrac{a}{P}\right\rfloor=a\bmod P, &\dfrac{a}{P}-\dfrac{a\lambda}{2^k}\ge\left\lfloor\dfrac{a}{P}\right\rfloor\\\\
a-P\left(\left\lfloor\dfrac{a}{P}\right\rfloor-1\right)=a\bmod P+P,&\dfrac{a}{P}-\dfrac{a\lambda}{2^k}<\left\lfloor\dfrac{a}{P}\right\rfloor
\end{cases}
$$

那么实现的时候随便特判一下即可。

现在想一下如何保证 $\dfrac{a\lambda}{2^k}<1$，容易发现 $a$ 最大是 $P-1$，那么 $2^k\ge P$ 即可，为了方便我们一般取 $k=64$（`unsigned long long` 的最大值加一），注意会爆 `unsigned long long`，所以中间要转化成 128 位无符号整数 `__uint128_t`。

可以用一个结构体封装一下，然后重载一下 `%` 符号。

/// details | 参考代码
    open: False
    type: success

```cpp
#define ull unsigned long long 
#define ui128 __uint128_t
struct Barrett{
    ull d;ui128 m;
    void init(ull _d){
        d=_d,m=(((ui128)(1)<<64)/d);
    }
}mod;
ull operator %(ull a,Barrett mod){
    ull w=(mod.m*a)>>64;w=a-w*mod.d;
    if(w>=mod.d)w-=mod.d;return w;
}
```

///