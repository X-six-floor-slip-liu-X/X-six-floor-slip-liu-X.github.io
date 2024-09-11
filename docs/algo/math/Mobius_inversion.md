---
comments: true
---

# 狄利克雷卷积与莫比乌斯反演入门

学之前感觉很神秘啊，但其实类似于容斥。

## 符号与约定

约定 $a\mid b$ 表示存在整数 $k$，使得 $ak=b$，同理有 $a\nmid b$，表示不存在那样的整数 $k$。

通常用 $\prod_i p_i^{c_i}$ 表示一个数的质因数分解，如果同时使用 $p,c$ 而不表示质因数分解会特殊说明。

## 狄利克雷卷积与数论函数相关概念

数论函数指定义域是正整数域 $\mathbb{Z}^+$，陪域（大概能理解为值域的超集）是复数域 $\mathbb{C}$ 的函数（但常见的的数论函数值域一般都是整数集合的子集）。它们叫数论函数貌似是因为数论非常喜欢研究这样的函数。

### 狄利克雷卷积

狄利克雷卷积是一种定义在数论函数上的运算，$f$ 和 $g$ 的狄利克雷卷积通常用 $f\ast g$ 来表示（就是星号 `\ast`，不是点 `\cdot`）。有 $(f\ast g)(n)=\sum_{xy=n}f(x)g(y)$，或者写作 $(f\ast g)(n)=\sum_{d\mid n}f(d)g(\frac{n}{d})$。

它有几乎和乘法一样的性质：

- **交换律**：$f\ast g=g\ast f$，显然。
- **结合律**：$(f\ast g)\ast h=f\ast (g\ast h)$，证明非常简单，因为 $\left((f\ast g)\ast h\right)(n)=\sum_{xyz=n}f(x)g(y)h(z)$，与运算顺序无关。
- **分配率**：$(f+g)\ast h=f\ast h+g\ast h$，其中加法表示单点函数值相加。证明也考虑套定义，略过。

以及一个神秘性质（虽然乘法也有这个性质，但证明方法不太一样）：

- $f=g\Leftrightarrow f\ast h=g\ast h$，其中数论函数 $h(x)$ 要满足 $h(1)\ne 0$。

/// admonition | 证明
    type: info

充分性显然，考虑证必要性。假设 $f\ne g$，那么必定存在一个最小的正整数 $x$，使得 $f(x)\ne g(x)$。

设 $r=f\ast h-g\ast h=(f-g)\ast h$，那么 $r(x)=\sum_{d\mid x}(f(d)-g(d))h(\frac{x}{d})$。

由于 $x$ 最小，所以所有小于 $x$ 的 $d$ 均满足 $f(d)=g(d)$，于是上式 $=(f(x)-g(x))h(1)\ne 0$，说明 $(f\ast h)(x)\ne (g\ast h)(x)$，则 $f\ast h\ne g\ast h$，与条件矛盾。故必要性成立。

然后你也能看出为什么要求 $h(1)\ne 0$。

///

### 积性函数与狄利克雷逆

在狄利克雷卷积意义下，有很多特殊函数：

- **元函数**：通常用 $\varepsilon$（`\varepsilon`）表示，$\varepsilon(n)=[n=1]$，对于任意数论函数 $f$ 均有 $\varepsilon\ast f=f$，原因带入定义证即可。
- **恒等函数**：通常用大写字母 $I$ 表示，$I(n)=1$。
- **单位函数**：$id(n)=n$，其实是幂函数（$id^x(n)=n^x$）的特殊情况，但也就它比较常用。

这些函数都是**完全积性函数**：

$f$ 是完全积性函数当且仅当对于任意整数 $a,b$，均有 $f(ab)=f(a)f(b)$。

那么既然有完全积性函数，那显然会定义有**积性函数**：

$f$ 是积性函数当且仅当对于任意整数 $a,b$ 满足 $\gcd(a,b)=1$，有 $f(ab)=f(a)f(b)$。

显然地，完全积性函数是积性函数的子集。

另外，由于单位元的存在，我们可以对数论函数定义狄利克雷逆。通常用 $f^{-1}$ 表示 $f$ 的狄利克雷逆，意义是 $f\ast f^{-1}=f^{-1}\ast f=\varepsilon$，根据 $f=g\Leftrightarrow f\ast h=g\ast h$ 的性质，同一个数论函数的狄利克雷逆唯一。

有一个性质，当且仅当 $f(1)\ne 0$ 时 $f$ 存在狄利克雷逆 $f^{-1}$。

/// admonition | 证明
    type: info

首先必要性比较简单，考虑否命题（即逆命题的逆否命题，若它为真那么逆命题为真，必要性成立）是当 $f(1)=0$，不存在狄利克雷逆。因为 $\varepsilon(1)=(f\ast f^{-1})(1)=f(1)f^{-1}(1)=1$，当 $f(1)=0$，这个方程无解。

然后考虑充分性，考虑从小到大构造 $f^{-1}$，注意到 $(f\ast f^{-1})(x)=\sum_{d\mid x}f(d)f^{-1}(\frac{x}{d})$ 中，除了 $d=1$ 都是确定的，那么构造如下：

$$
f^{-1}(x)=\frac{\varepsilon(x)-\sum_{d\mid x}^{d\ne 1}f(d)f^{-1}(\frac{x}{d})}{f(1)}
$$

这样就满足 $(f\ast f^{-1})(x)=\varepsilon(x)$ 了。显然当 $f(1)\ne 1$ 时均能构造出来，并且直接暴力跑复杂度甚至是 $O(n\log n)$（调和级数）的。

///

/// admonition | 积性函数的性质
    type: example

若 $f$ 是一个积性函数：

- $f(1)=1$ 或 $f(1)=0$。

证明简单，因为 $f(1)=f(1)f(1)$，但是若 $f(1)=0$ 那么整个函数全是 $0$，就没什么讨论的价值了。下文默认 $f(1)=1$。

- $f(n)=\prod_i f(p_i^{c_i})$。

显然，因为所有质因子均互质，套定义即可。

- 若 $f,g$ 均为积性函数，那么 $h=f\ast g$ 为积性函数。

证明考虑套定义，对于任意两个正整数 $a,b$ 满足 $\gcd(a,b)=1$，均有：

$$
\begin{aligned}
h(a)h(b)
&=\sum_{c\mid a}f(c)g(\frac{a}{c})\sum_{d\mid b}f(d)g(\frac{b}{d})\\\\
&=\sum_{c\mid a}\sum_{d\mid b}f(cd)g(\frac{ab}{cd})\\\\
&=\sum_{e|ab}f(e)g(\frac{ab}{e})\\\\
&=h(ab)
\end{aligned}
$$

- 若 $f$ 为积性函数，则 $f^{-1}$ 是积性函数。

考虑数学归纳法。首先对于 $xy=1$，显然有 $f^{-1}(1)=f^{-1}(1)f^{-1}(1)$。对于 $xy>1,\gcd(x,y)=1$，假设对于所有 $\gcd(n,m)=1,nm<xy$ 均有 $f^{-1}(nm)=f^{-1}(n)f^{-1}(m)$，那么：

$$
\begin{aligned}
f^{-1}(nm)
&=\frac{\varepsilon(nm)-\sum_{d\mid nm}^{d\ne 1}f(d)f^{-1}(\frac{nm}{d})}{f(1)}\\\\
&=-\sum_{d\mid nm}^{d\ne 1}f(d)f^{-1}(\frac{nm}{d})\\\\
&=-\sum_{a\mid n,b\mid m}^{ab\ne 1}f(ab)f^{-1}(\frac{nm}{ab})\\\\
&=-\sum_{a\mid n,b\mid m}^{ab\ne 1}f(a)f(b)f^{-1}(\frac{n}{a})f^{-1}(\frac{m}{b})\\\\
&=f(1)f(1)g(n)g(m)-\sum_{a\mid n,b\mid m}f(a)f(b)f^{-1}(\frac{n}{a})f^{-1}(\frac{m}{b})\\\\
&=f(1)f(1)g(n)g(m)-\sum_{a\mid n}f(a)f^{-1}(\frac{n}{a})\sum_{b\mid m}f(b)f^{-1}(\frac{m}{b})\\\\
&=f(1)f(1)g(n)g(m)-(f\ast f^{-1})(n)(f\ast f^{-1})(m)\\\\
&=f^{-1}(n)f^{-1}(m)-\varepsilon(n)\varepsilon(m)\\\\
&=f^{-1}(n)f^{-1}(m)
\end{aligned}
$$

其中最后一步能直接去掉 $\varepsilon(n)\varepsilon(m)$ 是因为 $n,m$ 中至少一个不为 $1$。

得证。

///

现在我们已经对狄利克雷卷积以及积性函数有了一点点了解了，我们来讨论一些非常特殊的积性函数。

## 莫比乌斯反演

芜湖！进入正题。

考虑很多时候可能存在 $F=f\ast I$ 的关系，并且 $F$ 是好求的而 $f$ 不好求。通过 $f$ 求 $F$ 是简单的，可以 $O(n\log n)$ 暴力，那根据 $F$ 求 $f$ 呢？根据反演的套路，可以考虑用 $f=F\ast I^{-1}$ 来求出 $f$。因为 Möbius 把 $I^{-1}$ 命名为 $\mu$（莫比乌斯函数），所以这样的反演叫莫比乌斯反演（虽然有时候用其它函数的狄利克雷逆做的反演也叫莫比乌斯反演）。

首先显然地，$\mu$ 是一个积性函数。对于积性函数通常先考虑它在质数的次幂时的表现，那么我们先考虑 $\mu(p^k)$，其中 $p$ 是质数。

对于 $k=0$ 是平凡的，$\mu(1)=1$。

当 $k=1$ 时，$\varepsilon(p)=\sum_{d\mid p}I(d)\mu(\frac{p}{d})=I(1)\mu(p)+I(p)\mu (1)=0$，则 $\mu(p)=-1$。

当 $k>1$ 时，$\sum_{i=0}^kI(p^i)\mu(p^{k-i})=0$，其中有两个值已经确定了，即 $I(p^{k-1})\mu(p)=-1,I(p^k)\mu(1)=1$，用数学归纳法易证当 $k>1$ 时，$\mu(p^k)=0$，具体细节略。

那么对于其他 $\mu(n)$，考虑把 $n$ 拆成 $\prod_i p_i^{c_i}$，由于 $\mu$ 时积性函数可以直接把对应函数值相乘，于是有：

$$
\mu(n)=\begin{cases}
0&,\exists p,p^2\mid n\\\\
(-1)^k&,\text{$k$ 是 $n$ 的质因子个数}
\end{cases}
$$

于是就能用欧拉筛 $O(n)$ 地求 $\mu$，或者 $O(\sqrt{n})$ 地单点求 $\mu$ 了。

然后我们能得到两组简单的反演关系：

$$
F(n)=\sum_{d\mid n}f(d)
$$
$$
f(n)=\sum_{d\mid n}\mu(d)F(\frac{n}{d})
$$

这个显然。

以及：

$$
F(n)=\sum_{k}f(k)
$$
$$
f(n)=\sum_{k}\mu(k)F(nk)
$$

（其实就是把因数换成了倍数）

这个也比较好证，仿照子集反演的证法：

$$
\begin{aligned}
f(n)
&=\sum_{i}\mu(i)F(ni)\\\\
&=\sum_{i}\mu(i)\sum_{j}f(nij)\\\\
&=\sum_{k}f(nk)\sum_{d\mid k}\mu(k)\\\\
&=\sum_{k}f(nk)\varepsilon(k)\\\\
&=f(n)
\end{aligned}
$$

但这两个其实都没啥用，不如下面这个：

$$
\sum_{d\mid n}\mu(d)=[n=1]
$$

证明非常显然，左边是 $\mu \ast I=\varepsilon$，右边就是 $\varepsilon$。

常用变形：

$$
[n\mid m]\sum_{d\mid \frac{m}{n}}\mu(d)=[n=m]
$$

显然。

$$
\sum_{d\mid gcd(i,j)}\mu(d)=[\gcd(i,j)=1]
$$

这个就很常用了。

//// details | 例题：[UVA12888 Count LCM](https://www.luogu.com.cn/problem/UVA12888)
    open: False
    type: example

简单推一下：

$$
\begin{aligned}
&\sum_{i=1}^n\sum_{j=1}^m[\mathrm{lcm}(i,j)=ij]\\\\
=&\sum_{i=1}^n\sum_{j=1}^m[\gcd(i,j)=1]\\\\
=&\sum_{i=1}^n\sum_{j=1}^m\sum_{d\mid gcd(i,j)}\mu(d)\\\\
=&\sum_{d=1}^{\min(n,m)}\sum_{d\mid i}^{i\le n}\sum_{d\mid j}^{j\le m}\mu(d)\\\\
=&\sum_{d=1}^{\min(n,m)}\mu(d)\sum_{d\mid i}^{i\le n}\sum_{d\mid j}^{j\le m}1\\\\
=&\sum_{d=1}^{\min(n,m)}\mu(d)\left\lfloor\frac{n}{d}\right\rfloor\left\lfloor\frac{m}{d}\right\rfloor\\\\
\end{aligned}
$$

然后就能整除分块 $O(\sqrt{\min(n,m)})$ 做了。

/// details | 参考代码
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define mem(a,b) memset(a,b,sizeof(a))
#define forup(i,s,e) for(i64 i=(s);i<=(e);i++)
#define fordown(i,s,e) for(i64 i=(s);i>=(e);i--)
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
using i64=long long;
#define gc getchar()
inline i64 read(){
    i64 x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<3)+(x<<1)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const i64 N=1e6+5;
i64 n,m,mu[N],vv[N],sum[N];
vector<i64> pri;
void init(i64 n){
	mu[1]=1;
	forup(i,2,n){
		if(!vv[i]){
			mu[i]=-1;
			pri.push_back(i);
		}
		for(auto j:pri){
			if(1ll*i*j>n) break;
			vv[i*j]=1;
			if(!(i%j)){
				mu[i*j]=0;
                break;
			}else{
				mu[i*j]=-mu[i];
			}
		}
	}
	forup(i,1,n){
		sum[i]=sum[i-1]+mu[i];
	}
}
signed main(){
	i64 t=read();
	init(1000000);
	while(t--){
		n=read();m=read();
		i64 res=0;
		for(i64 l=1,r=1;l<=min(n,m);l=r+1){
			r=min(n/(n/l),m/(m/l));
			res+=1ll*(sum[r]-sum[l-1])*(m/l)*(n/l);
		}
		printf("%lld\n",res);
	}
}
```

然后用类似的套路还能过 [P2158 [SDOI2008] 仪仗队](https://www.luogu.com.cn/problem/P2158)（下标从 $0$ 开始，容易发现除去 $(0,1),(1,0)$ 两个点以外和那个问题是等价的）[P3455 [POI2007] ZAP-Queries](https://www.luogu.com.cn/problem/P3455)（只算 $d$ 的倍数即可）。

///

////

### 欧拉函数

欧拉函数 $\varphi$（`\varphi`）也是一个重要的数论函数函数，$\varphi(n)=\sum_{i=1}^{n-1}[\gcd(n,i)=1]$。通项公式为 $\varphi(n)=n\prod_i\frac{p_i-1}{p_i}$，感性理解考虑每 $p_i$ 个数就有一个与 $n$ 不互质，理性证明网上很好找就不展开了。

显然 $\varphi$ 是一个积性函数，因为若 $a,b$ 互质，那么 $a,b$ 没有共同质因子，带一下通项公式容易发现 $\varphi(ab)=\varphi(a)\varphi(b)$。

然后如果要求 $\varphi$，可以 $O(\sqrt{n})$ 暴力带通项公式或者线性筛求出，此处不展开。

有一个显然的结论，$\sum_{i=1}^n\sum_{j=1}^n[\gcd(i,j)=1]=2\left(\sum_{i=1}^n\varphi(i)\right)-1$，容易发现 $\sum_{i=1}^n\sum_{j=1}^i[\gcd(i,j)=1]=\sum_{i=1}^n\varphi(i)$，这就枚举了所有 $i\ge j$ 的情况。直接互换 $i,j$ 位置会多算所有 $i=j$ 的情况，只有 $i=j=1$ 会有贡献，于是 $-1$。

然后有一个非常厉害的结论：$\varphi \ast I=id$。

/// admonition | 证明
	type: note

由于 $id$ 是积性函数，只需要考虑 $n=p^c$ 的情况。

那么 $(\varphi \ast I)(p^c)=\sum_{k=0}^c\varphi(p^k)$。

由于 $\varphi(p^c)=p^c\cdot \frac{p-1}{p}=p^{c-1}\cdot (p-1)$，那么后面就是一个等比数列求和。

于是有：

$$
\begin{aligned}
\sum_{k=0}^c\varphi(p^k)
&=1+\sum_{k=1}^c p^{k-1}\cdot (p-1)\\\\
&=1+\sum_{k=0}^{c-1} p^k\cdot (p-1)\\\\
&=1+\frac{(p-1)(p^c-1)}{p-1}\\\\
&=p^c
\end{aligned}
$$

得证。

///

既然已经看到了 $I$，那么就应该反应出 $\mu$，于是有 $\varphi=\mu \ast id$。

这东西就非常有用了，比如可以用来求 $\sum_{i=1}^n\sum_{j=1}^m\gcd(i,j)$（假设 $n\le m$）。

$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^m\gcd(i,j)
&=\sum_{t=1}^nt\sum_{i=1}^{\left\lfloor\frac{n}{t}\right\rfloor}\sum_{i=1}^{\left\lfloor\frac{m}{t}\right\rfloor}[\gcd(i,j)=1]\\\\
&=\sum_{t=1}^nt\sum_{d=1}^{\left\lfloor\frac{n}{t}\right\rfloor}\mu(d)\left\lfloor\frac{n}{dt}\right\rfloor\left\lfloor\frac{m}{dt}\right\rfloor\\\\
&=\sum_{t=1}^nt\sum_{t\mid T}^{T\le n}\mu(\frac{T}{t})\left\lfloor\frac{n}{T}\right\rfloor\left\lfloor\frac{m}{T}\right\rfloor\\\\
&=\sum_{T=1}^{n}\left\lfloor\frac{n}{T}\right\rfloor\left\lfloor\frac{m}{T}\right\rfloor\sum_{t\mid T} id(t)\mu(\frac{T}{t})\\\\
&=\sum_{T=1}^{n}\left\lfloor\frac{n}{T}\right\rfloor\left\lfloor\frac{m}{T}\right\rfloor\varphi(T)
\end{aligned}
$$

这真是太酷了。

然后还有一些比较显然的，此处不展开了。