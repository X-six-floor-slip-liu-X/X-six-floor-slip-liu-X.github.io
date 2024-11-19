---
comments: true
---

# 快速莫比乌斯变换，快速沃尔什变换

这两个算法用于处理位运算卷积，即 $h(x)=\sum_{a\otimes b=x}f(a)g(b)$，其中 $\otimes$ 表示按位与，按位或或按位异或。

## 快速莫比乌斯变换

前置知识：[高维前缀和](../dp/SOSDP.md)

快速莫比乌斯变换用于处理与卷积和或卷积。

对于 $h(x)=\sum_{a\;\mathrm{or}\; b=x}f(a)g(b)$

具体来说，设 $\hat{f}(x)=\sum_{y\;\mathrm{or}\; x=x}f(y)$，我们发现：

$$
\begin{aligned}
\hat{f}(x)\times\hat{g}(x)
&=\left(\sum_{y\;\mathrm{or}\; x=x}f(y)\right)\times \left(\sum_{y\;\mathrm{or}\; x=x}g(y)\right)\\\\
&=\sum_{y\;\mathrm{or}\; x=x}\sum_{z\;\mathrm{or}\; x=x} f(y)g(z)\\\\
&=\sum_{(y\;\mathrm{or}\; z)\;\mathrm{or}\; x=x} f(y)g(z)\\\\
&=\hat{h}(x)
\end{aligned}
$$

于是我们高维前缀和得到 $\hat{f},\hat{g}$，把 $\hat{f},\hat{g}$ 的点值相乘得到 $\hat{h}$，再做高维差分即可得到 $h$。

同理对于“与”的情况，我们改成高维后缀和和高维后缀差分即可。

## 快速沃尔什变换

FWT 也能处理与卷积和或卷积，但是显然不如 FMT 好写。所以我们只说异或卷积。

做法仍然是构造变换 $f\to \hat{f}$，我们希望有 $\hat{h}=\hat{f}\times \hat{h}$。

定义 $x\odot y$ 表示 $\mathrm{popcount}(x\;\mathrm{and}\; y)\bmod 2$，其中 $\mathrm{popcount}(x)$ 表示 $x$ 的二进制位数。

我们发现，$(x\odot y)\;\mathrm{xor}\;(x\odot z)=x\odot(y\;\mathrm{xor}\; z)$，注意到异或后 $\mathrm{popcount}$ 的奇偶性等于 $\mathrm{popcount}$ 奇偶性的异或，而 $x\odot y$ 相当于只保留 $y$ 在 $x$ 中 $1$ 处的位数，那么这个的证明是显然的。

我们设 $\hat{f}(x)=\left(\sum_{x\odot y=0}f(y)\right)-\left(\sum_{x\odot y=1}f(y)\right)$ 则：

$$
\begin{aligned}
\hat{f}(x)\times \hat{g}(y)
&=\left(\sum_{x\odot y=0}f(y)-\sum_{x\odot y=1}f(y)\right)\times \left(\sum_{x\odot y=0}g(y)-\sum_{x\odot y=1}g(y)\right)\\\\
&=\left(\sum_{x\odot y=0}f(y)\sum_{x\odot z=0}g(z)+\sum_{x\odot y=1}f(y)\sum_{x\odot z=1}g(z)\right)-\left(\sum_{x\odot y=0}f(y)\sum_{x\odot z=1}g(z)+\sum_{x\odot y=1}f(y)\sum_{x\odot z=0}g(z)\right)\\\\
&=\sum_{(x\odot y)\;\mathrm{xor}\;(x\odot z)=0}f(y)g(z)-\sum_{(x\odot y)\;\mathrm{xor}\;(x\odot z)=1}f(y)g(z)\\\\
&=\sum_{x\odot (y\;\mathrm{xor}\;z)=0}f(y)g(z)-\sum_{x\odot (y\;\mathrm{xor}\;z)=1}f(y)g(z)\\\\
&=\hat{h}(x)
\end{aligned}
$$

那么我们应该怎么实现 $f\to\hat{f}$ 和 $\hat{f}\to f$ 的变换呢？考虑一个类似于 FFT 的分治：

假设我们已经处理完了前 $i$ 位（即每个数只考虑和它后面完全相同的数，并且计算 $\odot$ 时只考虑前 $i$ 位）得到了 $\hat{f_i}(x)$，现在有两个数 $x,y$，前 $i$ 位完全相同，后面也完全相同，只有第 $i$ 位 $x$ 为 $0$ 而 $y$ 为 $1$。

因为 $x$ 第 $i$ 位为 $0$，那么它和所有数 $\odot$ 的值应该都和只考虑前 $i-1$ 位时相同，则 $\hat{f_{i}}(x)=\hat{f_{i-1}}(x)+\hat{f_{i-1}}(y)$。因为 $y$ 第 $i$ 位为 $1$，那么它和第 $i$ 位为 $0$ 的数 $\odot$ 答案不变，但是和这一位为 $1$ 的 $\odot$ 答案反转，于是 $\hat{f_{i}}(y)=\hat{f_{i-1}}(x)-\hat{f_{i-1}}(y)$。

逆运算是简单的，有 $\hat{f_{i-1}}(x)=\frac{\hat{f_{i}}(x)+\hat{f_{i}}(y)}{2},\hat{f_{i-1}}(y)=\frac{\hat{f_{i}}(x)-\hat{f_{i}}(y)}{2}$， ~~小学奥数和差问题~~ 。

/// details | [模板题](https://www.luogu.com.cn/problem/P4717) 参考代码 
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
using pii=pair<int,int>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
int read(){
    int x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const int N=1<<17|5,inf=0x3f3f3f3f,mod=998244353,inv2=(mod+1)/2;
int n,m,a[N],b[N];
int f[N],g[N];
void getor(int *f,bool flag){
    forup(i,0,n-1){
        forup(j,0,m-1){
            if((j>>i)&1){
                if(flag){
                    (f[j]+=f[j^(1<<i)])%=mod;
                }else{
                    f[j]=(f[j]+mod-f[j^(1<<i)])%mod;
                }
            }
        }
    }
}
void getand(int *f,bool flag){
    forup(i,0,n-1){
        forup(j,0,m-1){
            if(!((j>>i)&1)){
                if(flag){
                    (f[j]+=f[j^(1<<i)])%=mod;
                }else{
                    f[j]=(f[j]+mod-f[j^(1<<i)])%mod;
                }
            }
        }
    }
}
void getxor(int *f,bool flag){
    forup(i,0,n-1){
        int p=(1<<i);
        for(int j=0;j<m;j+=(p<<1)){
            forup(k,0,p-1){
                int x=f[j+k],y=f[j+k+p];
                if(flag){
                    f[j+k]=(x+y)%mod;
                    f[j+k+p]=(x+mod-y)%mod;
                }else{
                    f[j+k]=1ll*(x+y)*inv2%mod;
                    f[j+k+p]=1ll*(x+mod-y)*inv2%mod;
                }
            }
        }
    }
}
signed main(){
    n=read();m=1<<n;
    forup(i,0,m-1) a[i]=read();
    forup(i,0,m-1) b[i]=read();
    forup(i,0,m-1){f[i]=a[i];g[i]=b[i];}
    getor(f,1);getor(g,1);
    forup(i,0,m-1) f[i]=1ll*f[i]*g[i]%mod;
    getor(f,0);
    forup(i,0,m-1) printf("%d ",f[i]);
    puts("");
    forup(i,0,m-1){f[i]=a[i];g[i]=b[i];}
    getand(f,1);getand(g,1);
    forup(i,0,m-1) f[i]=1ll*f[i]*g[i]%mod;
    getand(f,0);
    forup(i,0,m-1) printf("%d ",f[i]);
    puts("");
    forup(i,0,m-1){f[i]=a[i];g[i]=b[i];}
    getxor(f,1);getxor(g,1);
    forup(i,0,m-1) f[i]=1ll*f[i]*g[i]%mod;
    getxor(f,0);
    forup(i,0,m-1) printf("%d ",f[i]);
    puts("");
}
```

////