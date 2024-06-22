---
comments: true
---

# FFT（快速傅里叶变换）与 NTT（快速数论变换）

多项式全家桶其之一。

[参考资料1](https://xxeray.gitlab.io/post/fast-fourier-transform/) [参考资料2](https://oi.wiki/math/poly/fft/)

前置知识：

复数（请移步高中必修二或 [OI wiki](https://oi-wiki.org/math/complex/)）。

初中范围的多项式。

## 符号与约定

为了好看（以及好写），用 $\langle a_0,a_1,a_2,\dots a_n\rangle$ 表示一个有序 $n$ 元组，也用这个表示数列（某种意义上，数列也是有序 $n$ 元组吧）。

$a\mid b$ 表示存在整数 $k$ 使得 $ak=b$。

## 单位根

前置知识：复数。

众所周知，任意一元 $n$ 次方程必有 $n$ 个根。只是初中范围（以及高中“复数”这一章以外）内只研究实根，而在遇到 $\sqrt{-a}$ 之类的东西都会直接省去。实际上这是能产生复数根的。

考虑 $x^n=1$ 这个方程。由于复数的运算法则（模长相乘，辐角相加），显然这 $n$ 个解恰好把复平面单位圆均分成 $n$ 份（即辐角分别为 $\frac{2k\pi}{n},k\in[0,n-1]$）。我们称 $x^n=1$ 的根为 $n$ 次单位根，记除去 $1$ 之外辐角最小的 $n$ 次单位根为 $\omega_n$（`\omega_n`），显然其它单位根就是 $\omega_n^i$

单位根有三个比较显然的引理：

1. 消去引理：$\omega_{kn}^{km}=\omega_n^m$，根据单位根几何意义显然。
2. 折半引理：$\left(\omega_{n}^{k+\frac{n}{2}}\right)^2=\left(\omega_{n}^k\right)^2=\omega_{n}^{2k}=\omega_{\frac{n}{2}}^k$，这个只在 $n$ 为偶数时成立，带消去引理即可。
3. 求和引理：$\sum_{i=0}^{n-1} \left(\omega_n^k\right)^i = 0$，这玩意显然就等于 $\sum_{i=0}^{n-1} \omega_n^i$，是一个等比数列求和，答案即为 $\frac{\omega_n^n-1}{\omega_n^1-1}=\frac{1-1}{\omega_n^1-1}==0$。

另外根据三角函数基础，显然 $\omega_n=\cos(\frac{2\pi}{n})+\sin(\frac{2\pi}{n})i$。

## 多项式

前置知识：初中范围的多项式。

此处只讨论一元 $n$ 次多项式。即 $F(x)=\sum_{i=0}^na_ix^i$。

多项式有两种表示方法：

1. 系数表示：即列举出每一项的系数 $\langle a_0,a_1,\dots a_n\rangle$，其中 $a_i$ 为 $x^i$ 项的系数。
2. 点值表示：根据经典结论，$n+1$ 个互不相同的点就能确定一个一元 $n$ 次多项式，那么可以用 $\langle(x_0,F(x_0)),(x_1,F(x_1)),(x_2,F(x_2)),\dots (x_{n+1},F(x_{n+1}))\rangle$ 唯一确定一个多项式。并且这个神秘性质在定义域取到复数域，甚至是某个质数的剩余系时仍然成立，我证不来。

## FFT

### 相关概念

- 离散傅里叶变换（Discrete Fourier Transform，DFT），将系数表示转化为某个点值表示的方法（实际定义比较复杂，涉及到一些信号学知识，此处可以这样理解）。
- 快速傅里叶变换（即 快速（离散）傅里叶变换，Fast (Discrete) Fourier Transform，FFT），复杂度较低的 DFT。

下文的 $n$ 均指多项式次数 $+1$（因为 $0$ 次项也要占一个位置，不这样搞会导致有一堆奇怪的 $+1$），称这个数为多项式“项数”。

### 算法引入

假如我们要算多项式的积（$H(x)=G(x)\times F(x)$），直接用系数表示做需要 $O(n^2)$ 做。

但是假如我们知道了 $G$ 和 $F$ 的点值表示，根据 $H(x)=G(x)\times F(x)$，就能 $O(n)$ 求出 $H$ 的点值表示（容易发现 $G,F,H$ 的项数不同，其中 $F$ 为 $n$ 项，$G$ 为 $m$ 项，$H$ 为 $n+m-1$ 项。一个办法是把三者都看成 $n+m$ 项式）。如果我们还有能从点值表示推出系数表示的方法，就能得到 $H(x)$ 的系数表示了。

并且容易发现多项式乘法就是对系数的卷积，那么这玩意还能用来快速求卷积。

### 算法流程

首先我们肯定不能随便找一些 $x_i$，这样显然不好优化。FFT 干的事情是先找到最小的 $k$ 满足 $2^k\ge n$，把 $n$ 项多项式视为一个 $2^k$ 项多项式，然后用 $\omega_{2^k}^i$ 作为 $x_i$ 来表示点值。下文会叙述为什么要这样搞。

首先 FFT 的核心思想是分治，对于一个多项式 $F$ 的系数表示 $A=\langle a_0,a_1,a_2,\dots a_{n-1}\rangle$（$n=2^k$），可以把它按奇偶性分成 $A_0=\langle a_0,a_2,a_4,\dots a_{n-2}\rangle$ 和 $A_1=\langle a_1,a_3,a_5,\dots a_{n-1}\rangle$，设这两个系数表示对应的多项式分别为 $F_0,F_1$，容易发现 $F(x)=F_0(x^2)+xF_1(x^2)$（显然）。于是就能想到用 $F_0,F_1$ 的点值表示来算出 $F$ 的点值表示。

但这里有个问题，$F_0,F_1$ 的点值表示都只有 $\frac{n}{2}$ 个点，这样似乎无论怎么搞都只能得到 $F$ 的 $\frac{n}{2}$ 个点。这时候你就知道为什么要用单位根作为 $x_i$ 了。

取 $k$ 为某个 $[0,\frac{n}{2})$ 之中的整数，记 $k'=k+\frac{n}{2}$，容易发现：

$$
\begin{aligned}
F(\omega_n^k)&=F_0\left(\left(\omega_n^k\right)^2\right)+\omega_n^kF_1\left(\left(\omega_n^k\right)^2\right)&=F_0\left(\omega_{\frac{n}{2}}^k\right)+\omega_n^kF_1\left(\omega_{\frac{n}{2}}^k\right)\\\\
F(\omega_n^{k'})&=F_0\left(\left(\omega_n^{k'}\right)^2\right)+\omega_n^{k'}F_1\left(\left(\omega_n^{k'}\right)^2\right)&=F_0\left(\omega_{\frac{n}{2}}^k\right)-\omega_n^kF_1\left(\omega_{\frac{n}{2}}^k\right)
\end{aligned}
$$

其中第二行是因为 $\omega_n^{k+\frac{n}{2}}=-\omega_n^k$。

然后就能用 $F_0,F_1$ 的点值表示推出 $F$ 的点值表示了。对于 $n=1$ 的边界条件，显然 $F(\omega_1^0)=F(1)=a_0x=a_0$，于是就有一个简单的递归写法了。需要封装复数的加减乘法。

### 非递归写法

容易发现这玩意常数很大（因为每次需要把序列按奇偶性分开，可能需要开一些辅助数组，内存访问就不太连续），而这种 $2^k$ 每次都能恰好地把序列分成两半，这和 zkw 线段树十分类似，可以考虑是否存在非递归写法。

容易发现，如果把它看做 zkw 的结构，我们只需要求出所有 $n=1$ 在递归树上对应的下标，其它就能直接倒序循环求出。

以 $n=8$ 为例，考虑递归过程：

```
a[0]   a[1]   a[2]   a[3]   a[4]   a[5]   a[6]   a[7] 
a[0]   a[2]   a[4]   a[6] | a[1]   a[3]   a[5]   a[7] 
a[0]   a[4] | a[2]   a[6] | a[1]   a[5] | a[3]   a[7] 
a[0] | a[4] | a[2] | a[6] | a[1] | a[5] | a[3] | a[7] 
```

其实已经能发现一些端倪了，第 $i$ 层是按照二进制从下到上第 $i$ 为分开的，那么从二进制方向考虑每个点的下标：

```
0   4   2   6   1   5   3   7
000 100 010 110 001 101 011 111
```

容易发现这些东西的二进制表示翻转后就恰好是 $0\sim 7$，从每次“以当前最低位为依据分开”也能推出来。于是直接以这样的“逆二进制序”处理即可。

于是就能得到非递归写法了。

代码等会和后面的整合一下。

### 逆 FFT（IFFT）

根据 $F(\omega_{n}^i)=\sum_{j=0}^{n-1}a_j\omega_n^{ij}$，容易发现这玩意是个线性变换。

那么就能写成矩阵 $\mathbf{y}=\mathbf{a}\times \mathbf{V{\mathit{n}}}$，其中 $(\mathbf{V{\mathit{n}}})_{i,j}=\omega_{n}^{ij}$。

然后用神秘方法就能得到 $\mathbf{V_{\mathit{n}}}^{-1}$，使得 $\mathbf{a}=\mathbf{y}\times\mathbf{V_{\mathit{n}}}^{-1}$。

直接说结论 $(\mathbf{V_{\mathit{n}}}^{-1})_{i,j}=\frac{\omega_{n}^{-ij}}{n}$。 ~~因为我不会证~~

以及容易发现，$\omega_{n}^{-1}=\cos(\frac{2\pi}{n})+\sin(-\frac{2\pi}{n})i$（考虑 $\omega_{n}^{-1}\times \omega_{n}=1$，模长相乘辐角相加）。然后其余部分和 FFT 完全相同，最后除以 $n$（注意这个 “$n$” 是前文所说 $2^k$）即可。

/// [模板题](https://www.luogu.com.cn/problem/P3803)参考代码
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define forup(i,s,e) for(int i=(s),E123123123=(e);i<=E123123123;++i)
#define fordown(i,s,e) for(int i=(s),E123123123=(e);i>=E123123123;--i)
#define mem(a,b) memset(a,b,sizeof(a))
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void();
#endif
using namespace std;
using i64=long long;
using ld=long double;
#define gc getchar()
int read(){
	int x=0,f=1;char c;
	while(!isdigit(c=gc)) if(c=='-') f=-1;
	while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
	return x*f;
}
#undef gc
const int N=1<<21;
const ld pi=acosl(-1),eps=1e-7;
int n,m,p,w;
struct Complex{
	ld x,y;
	Complex(ld _x=0,ld _y=0):x(_x),y(_y){}
	Complex operator +(const Complex &r){return Complex{x+r.x,y+r.y};}
	Complex operator -(const Complex &r){return Complex{x-r.x,y-r.y};}
	Complex operator *(const Complex &r){return Complex{x*r.x-y*r.y,x*r.y+y*r.x};}
};
Complex f[N],g[N];
int rev[N];
void trans(Complex *f,int type){
	forup(i,0,p-1) if(i<rev[i]) swap(f[i],f[rev[i]]);
	for(int len=1;len<p;len<<=1){
		Complex wn(cosl(pi/len),type*sinl(pi/len));
		for(int i=0;i<p;i+=(len<<1)){
			Complex nw(1,0);
			forup(j,0,len-1){
				Complex x=f[i+j],y=nw*f[i+len+j];
				f[i+j]=x+y;
				f[i+len+j]=x-y;
				nw=nw*wn;
			}
		}
	}
	if(type==-1) forup(i,0,p-1) f[i].x/=p;
}
signed main(){
	n=read();m=read();
	++n;++m;
	forup(i,0,n-1) f[i].x=read();
	forup(i,0,m-1) g[i].x=read();
	p=1;w=0;
	while(p<n+m) p<<=1,++w;
	forup(i,0,p-1){
		rev[i]=(rev[i>>1]>>1)|((i&1)<<(w-1));
	}
	trans(f,1);
	trans(g,1);
	forup(i,0,p-1){
		f[i]=f[i]*g[i];
	}
	trans(f,-1);
	forup(i,0,n+m-2){
		if(fabs(f[i].x<eps)){
			printf("0 ");
		}else{
			printf("%.0Lf ",f[i].x);
		}
	}
}
```

///


## NTT

前置知识：数论基础。

事实上如果你真的写了 FFT 模板题就会发现有一堆精度问题，而且假如用来卷积这玩意不能取模。

于是就有人发明了没有精度问题（但是需要取模）的类似算法：快速数论变换（Number Theoretic Transform，NTT）。

很多 OIer 在第一次做需要取模的计数题时都会疑惑“为什么模数是 $998244353$”，就是因为有这个算法的存在。

$998244353$ 有非常好的性质：

- 它是质数，剩余系对乘法是封闭的。
- $\varphi(998244353)=998244352=2^{23}\times 7\times 17$
- 它有[原根](https://oi-wiki.org/math/number-theory/primitive-root/#_4)，其中一个是 $3$。

下文默认在模 $998244353$ 意义下进行。

原根（简要理解）就是说一个 $p$ 的简化剩余系内的数 $a$，若满足 $0\le n< \varphi(p)$，所有 $a^n\bmod p$ 互不相同（或者 $\forall 0<n<\varphi(p),a^n\not\equiv 1\pmod p$，这两个定义显然等价），就称 $a$ 是 $p$ 的原根。

考虑 FFT 中为什么要用 $\omega_n^i$ 作为点值表示的下标（下文记 $x_i$ 表示点值表示的第 $i$ 个下标）。就是因为它具有消去引理（$\omega_{kn}^{km}=\omega_n^m$）和折半引理（$\left(\omega_{n}^{k+\frac{n}{2}}\right)^2=\left(\omega_{n}^k\right)^2=\omega_{n}^{2k}=\omega_{\frac{n}{2}}^k$），那么原根有没有类似的性质呢？

记原根为 $g$（当 $p=998244353$ 时，$g=3$），又记 $g_n=g^{\frac{p-1}{n}}$（钦定 $n\mid p-1$）。于是有：

- $g_{kn}^{km}=g^{\frac{km(p-1)}{kn}}=g^{\frac{m(p-1)}{n}}=g_n^m$
- $\left(g_{n}^{k+\frac{n}{2}}\right)^2=g_n^{2k+n}=g^{\frac{(2k+n)(p-1)}{n}}=g^{\frac{2k(p-1)}{n}}+g^{p-1}=g^{\frac{2k(p-1)}{n}}=g_n^{2k}=g_{\frac{n}{2}}^k$

然后就能用这玩意做类似于 FFT 的操作了。

注意到一个问题，刚刚钦定了 $n\mid p-1$。

哎，您猜怎么着☝️🤓，$998244352$ 有 $2^{23}$ 的因子（$\approx 8.3\times 10^6$），可以把 $n$ 调整为 $2$ 的幂次就能做了。

注意到一点区别，从 $F_0,F_1$ 推到 $F$ 的式子（$k\in[0,\frac{n}{2}),k'=k+\frac{n}{2}$）：

$$
\begin{aligned}
F(g_n^k)&=F_0\left(\left(g_n^k\right)^2\right)+g_n^kF_1\left(\left(g_n^k\right)^2\right)&=F_0\left(g_{\frac{n}{2}}^k\right)+g_n^kF_1\left(g_{\frac{n}{2}}^k\right)\\\\
F(g_n^{k'})&=F_0\left(\left(g_n^{k'}\right)^2\right)+g_n^{k'}F_1\left(\left(g_n^{k'}\right)^2\right)&=F_0\left(g_{\frac{n}{2}}^k\right)+g^{\frac{p-1}{2}}g_n^kF_1\left(g_{\frac{n}{2}}^k\right)
\end{aligned}
$$

注意到 $g^{\frac{p-1}{2}}=\sqrt{g^{p-1}}=\sqrt{1}$，根据二次探测引理，$g^{\frac{p-1}{2}}=\pm 1$。但是若这玩意等于 $1$ 那么 $g$ 就不是原根了（和定义不符），所以这东西必定等于 $-1$（即 $p-1$）。

/// details | 参考代码
	open: False
	type: success

其它地方差不多，`inv3` 是 $3$ 的逆元。

```cpp
void trans(int *f,int type){
	forup(i,0,p-1) if(i<rev[i]) swap(f[i],f[rev[i]]);
	for(int len=1;len<p;len<<=1){
		int wn=ksm(type==1?3:inv3,(mod-1)/(len<<1));
		for(int i=0;i<p;i+=(len<<1)){
			int nw=1;
			forup(j,0,len-1){
				int x=f[i+j],y=1ll*nw*f[i+len+j]%mod;
				f[i+j]=(x+y)%mod;
				f[i+len+j]=(x-y+mod)%mod;
				nw=1ll*nw*wn%mod;
			}
		}
	}
	if(type==-1){
		int inv=ksm(p,mod-2);
		forup(i,0,p-1) f[i]=1ll*f[i]*inv%mod;
	}
}
```

///

## 任意模数 NTT

感觉没什么用就先不写了，留坑待补。