---
comments: true
---

# 高维前缀和与 SOSDP

## 前言

在某次模拟赛中，我打表推出了 T2 的式子，那是一个看起来很好维护的东西（我给你说你不知道经过一系列思维风暴后看到一个极其好维护的东西之后是什么心情），但是我浪费了 3h 没想出怎么维护，遂爆零。后来赛后发现它是一个叫高维前缀和的板子，前来学习。

## 引入

高维前缀和，又叫 SOSDP（Sum over Subsets dynamic programming，子集动态规划）它一般是用来解决子集类的求和问题（虽然也可以解决高维空间的求和问题，但是时空往往不允许）。

我们通常求前缀和是这样的（不会打伪代码，直接上 C++）：

一维：

/// details | code
    open: False

```cpp
int sum[N],a[N],n;

///input a[1...n]

forup(i,1,n){
    sum[i]=sum[i-1]+a[i];
}
```

///

而二维通常使用**容斥**解决：

/// details | code
    open: False

```cpp
int sum[N][N],a[N][N],n,m;

///input a[1...n][1...m]

forup(i,1,n){
    forup(j,1,m){
        sum[i][j]=sum[i-1][j]+sum[i][j-1]-sum[i-1][j-1]+a[i][j];
    }
}
```

///

三维类似，但是需要重新思考容斥方法，而且已经开始变得很**抽象**（本义）了：

/// details | code
    open: False

```cpp
int sum[N][N][N],a[N][N][N],n,m,w;

///input a[1...n][1...m][1...w]

forup(i,1,n){
    forup(j,1,m){
        forup(k,1,w){
            sum[i][j][k]=sum[i-1][j][k]+sum[i][j-1][k]+sum[i][j][k-1]-
                         sum[i-1][j-1][k]-sum[i-1][j][k-1]-sum[i][j-1][k-1];
        }
    }
}
```

///

然后四维……就不折磨自己了吧。

这样就和多次方程的求解一样，到了一定层数就不能拓展了……

吗？

我们考虑二维前缀和的另一种求法，把每一维分开求：

/// details | code
    open: True

```cpp
int sum[N][N],a[N][N],n,m;

///input a[1...n][1...m]

forup(i,1,n){
    forup(j,1,m){
        sum[i][j]=sum[i][j-1]+a[i][j];
    }
}
forup(i,1,n){
    forup(j,1,m){
        sum[i][j]+=sum[i-1][j];
    }
}
```

///

可以自己画图推一下正确性，这样求有两个好处：

1. 在一些不存在逆运算的场合（比如非质数模意义下的前缀积）可以求出前缀和。
2. 可以拓展到高维。

如何拓展到高维呢？比如我们先拓展到三维：

/// details | code
    open: False

```cpp
int sum[N][N][N],a[N][N][N],n,m,w;

///input a[1...n][1...m][1...w]

forup(i,1,n){
    forup(j,1,m){
        forup(k,1,w){
            sum[i][j][k]=sum[i-1][j][k]+a[i][j][k];
        }
    }
}
forup(i,1,n){
    forup(j,1,m){
        forup(k,1,w){
            sum[i][j][k]+=sum[i][j-1][k];
        }
    }
}
forup(i,1,n){
    forup(j,1,m){
        forup(k,1,w){
            sum[i][j][k]+=sum[i][j][k-1];
        }
    }
}
```

///

那么我们可以很容易的拓展到**任意维**：

/// details | code
    open: False

```cpp
forup(i1,1,n1){
    forup(i2,1,n2){
        ...
        sum[i1][i2][...]+=sum[i1-1][i2][...];
    }
}
forup(i1,1,n1){
    forup(i2,1,n2){
        ...
        sum[i1][i2][...]+=sum[i1][i2-1][...];
    }
}
...
```

///

差不多这样。

## 子集

但是这和子集有什么关系呢？

考虑这么一个例题：

> 给定序列 $a_{1...n}$，对于所有 $j\in[0,2^k-1]$，求 $\sum_{i\subseteq j}a_i$ 的值，其中 $\subseteq$ 表示二进制下 $1$ 的集合的包含关系。

首先可以暴力枚举子集 $O(2^k3^k)$ 求解，但是这道题其实可以用高维前缀和 $O(2^k)$ 求解。

容易发现，二进制下 $1$ 的集合的包含关系其实可以看做 $0,1$ 的偏序关系，那么子集求和问题就可以看做 $k$ 维下，每一维下标为 $[0,1]$ 的高维前缀和。

那么代码就很好想了，最外层循环从 $0$ 到 $k-1$ 枚举 $i$ 表示当前转移的维度，然后内层从 $0$ 到 $2^k-1$ 枚举 $j$ 等价于高维前缀和中对每一维的枚举，然后考虑若 $j$ 的第 $i$ 位为 $1$ 那么它在这一维就存在一个前驱 $0$。

/// details | 参考代码
    open: True

```cpp
int a[1<<k],sum[1<<k];

//input

forup(i,0,(1<<k)-1){
    sum[i]=a[i];
}
forup(i,0,k-1){
    forup(j,0,(1<<k)-1){
        if(j&(1<<i)) sum[j]+=sum[j^(1<<i)];
    }
}
```

///

然后我们就能 $O(2^k)$ 解决这个问题啦。

然后还能拓展到超集求和，把 $01$ 反过来考虑即可。

另外值得一提的是，和前缀和类似，对于所有具有结合律和交换律的信息都能使用高维前缀和。

## 例题

这个后面来补