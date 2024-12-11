---
comments: true
---

# k-D Tree

k-D Tree(KDT , k-Dimension Tree) 是一种可以**高效处理 $k$ 维空间信息**的数据结构。

在结点数 $n$ 远大于 $2^k$ 时，应用 k-D Tree 的时间效率很好（但还是根号级别的）。

在算法竞赛中，$k$ 通常等于 $2$，但其实更高维度也不是不能做，本文若无特殊说明默认 $k=2$。

## 建树

虽然 KDT 常用来维护高维信息，但它实际上是一种一维数据结构（而非树套树等二维数据结构）。

具体来说，KDT 具有二叉搜索树的结构，每个子树代表一个矩形（或者超长方体，下文仍以“矩形”称），维护这个矩形内的所有点的信息。

对于每个结点，我们用它将自己的子树分成两个不交的矩形，然后向下对两边的点集递归。这样 KDT 的深度就是 $\log n+O(1)$ 级别的。

/// admonition | 关于 KDT 是否 leafy
    type: note

通常来说，KDT 有两种常见写法：leafy 和 nodey。

顾名思义，leafy 指的是只在叶子结点维护点的信息（类似于线段树），而 nodey 指的是在每个结点都维护点的信息（类似于平衡树）。

两者常数相近，空间相近（事实上，通常 nodey 写法的空间大一点，时间小一点），但是 leafy 更好写一点。

刚刚说的“用它将自己的子树分成两不交的矩形”，在 leafy 时就是单纯分成两矩形，在 nodey 时就是找到一个点，去掉它之后再分成两矩形。

///

显然感觉上每次找中点是较优的。事实上，我们每一层轮流找 $k$ 个维度的中点分开（比如这一层找 $x$ 下一层找 $y$，依此类推），我们可以在后面的复杂度分析看到这为什么是对的。

注意到我们需要找到区间中点，直接排序每一层是 $O(n\log n)$ 的，但是有神秘做法可以 $O(n)$ 找到中点，并且将更小的扔到左边，更大的扔到右边。具体来说，类似于快速排序，每次找一个基准点，然后将所有点按大小放在它的左右两侧，再向其中一边递归。这个算法是 $O(n)$ 的。事实上，我们可以直接引用标准库中的 `nth_element(s+l,s+mid+1,s+r+1,cmp)` 将 $s$ 数组中排序后第 $mid-1$ 个数放在第 $mid-1$ 位，更小的放在左边，更大的放在右边。

这样我们就能 $O(n\log n)$ 建树了。

## 查询

查询通常是查询某矩形内既有结合律又有交换律的信息，比如区间和。

通常写法是记录每一棵子树每一维坐标的最小值/最大值，若和查询矩形 $R$ 相离就返回 $0$，若完全被 $R$ 包含返回区间和，否则向下递归。

关于复杂度问题，考虑每一个结点代表的区间只有以上三种情况，显然复杂度应与与 $R$ 相交但不被 $R$ 完全包含的区间数级别相同。

注意到这样的区间要么完全包含 $R$，要么和其相交但不包含，前者显然最多只有 $O(\log n)$ 个，因为树高是 $O(\log)$ 的，考虑后者。

注意到相交但不包含的区间必定有至少一条 $R$ 的边穿过，于是不妨统计 $R$ 的边穿过了多少个区间。

由于每一层是 $k$ 维轮流找中点，那么连续 $k$ 层必定把矩形每一维都分成了两半，我们不妨把这 $k$ 层放在一起看，于是一条边在 $k$ 层后至多穿过 $2^{k-1}$ 个区间，根据主定理，复杂度即为：

$$
T(n)=2^{k-1}T(\frac{n}{2^k})+O(1)=O(n^{1-\frac{1}{k}})
$$

在 $k=2$ 时就是 $O(\sqrt{n})$。

## 插入/删除

有时候我们需要在坐标系中插入一个点，但是直接向平衡树一样就不能保证树高恰为 $\log n+O(1)$ 了，这时候有一个神秘 Trick：二进制分组。

具体来说，我们维护 $\left\lceil\log n\right\rceil$ 棵 KDT（这里的 $n$ 是插入总量），第 $i$ 棵的大小为 $2^i$（从 $0$ 开始标号），每次向二进制进位一样，加点时向这个二进制数 $+1$，将期间进位的 $1$ 全部合并到下一位上，合并就是把所有点取出来重新建树。

容易发现一个点最多进位 $\log n$ 次，所以建树复杂度 $O(n\log^2 n)$。

查询在每棵树上查即可，复杂度还是 $O(n^{1-\frac{1}{k}})$。


### 例题

//// [洛谷 P4148 简单题](https://www.luogu.com.cn/problem/P4148)
    open: False
    type: note

就是板子。

/// details | 参考代码
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
#define msg(...) void()
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
const int N=2e5+5,inf=0x3f3f3f3f;
int n;
struct Node{
    int x,y,a;
};
int getp(bool ps,Node a){
    if(!ps) return a.x; 
    else    return a.y;
}
struct _2D_tree{
    int ls[N],rs[N],xl[N],xr[N],yl[N],yr[N],cntn;
    int px[N],py[N],val[N],sum[N];
    int root[18];
    vector<int> stk;
    int New(int x,int y,int a){
        int nw;
        if(stk.size()){
            nw=stk.back();stk.pop_back();
        }else{
            nw=++cntn;
        }
        px[nw]=x;py[nw]=y;val[nw]=a;
        ls[nw]=rs[nw]=0;
        xl[nw]=xr[nw]=px[nw];
        yl[nw]=yr[nw]=py[nw];
        sum[nw]=val[nw];
        return nw;
    }
    void Get(int id,vector<Node> &vec){
        vec.push_back(Node{px[id],py[id],val[id]});
        stk.push_back(id);
        if(ls[id]) Get(ls[id],vec);
        if(rs[id]) Get(rs[id],vec);
    }
    void PushUp(int id){
        xl[id]=xr[id]=px[id];
        yl[id]=yr[id]=py[id];
        sum[id]=val[id];
        if(ls[id]){
            xl[id]=min(xl[id],xl[ls[id]]);
            yl[id]=min(yl[id],yl[ls[id]]);
            xr[id]=max(xr[id],xr[ls[id]]);
            yr[id]=max(yr[id],yr[ls[id]]);
            sum[id]+=sum[ls[id]];
        }
        if(rs[id]){
            xl[id]=min(xl[id],xl[rs[id]]);
            yl[id]=min(yl[id],yl[rs[id]]);
            xr[id]=max(xr[id],xr[rs[id]]);
            yr[id]=max(yr[id],yr[rs[id]]);
            sum[id]+=sum[rs[id]];
        }
    }
    void Build(int &id,bool ps,vector<Node> &vec,int l,int r){
        if(l>=r) return;
        if(l+1==r){
            id=New(vec[l].x,vec[l].y,vec[l].a);
            PushUp(id);
            return;
        }
        int mid=(l+r)>>1;
        nth_element(vec.begin()+l,vec.begin()+mid+1,vec.begin()+r,[&](Node a,Node b){return getp(ps,a)<getp(ps,b);});
        id=New(vec[mid].x,vec[mid].y,vec[mid].a);
        Build(ls[id],!ps,vec,l,mid);
        Build(rs[id],!ps,vec,mid+1,r);
        PushUp(id);
    }
    void Update(int x,int y,int a){
        vector<Node> vec;
        vec.push_back(Node{x,y,a});
        int pl=0;
        while(root[pl]){
            Get(root[pl],vec);
            root[pl]=0;
            ++pl;
        }
        Build(root[pl],0,vec,0,vec.size());
    }
    int query(int id,int XL,int XR,int YL,int YR){
        if(XL>xr[id]||xl[id]>XR||YL>yr[id]||yl[id]>YR) return 0;
        if(XL<=xl[id]&&xr[id]<=XR&&YL<=yl[id]&&yr[id]<=YR) return sum[id];
        int res=0;
        if(XL<=px[id]&&px[id]<=XR&&YL<=py[id]&&py[id]<=YR) res+=val[id];
        if(ls[id]) res+=query(ls[id],XL,XR,YL,YR);
        if(rs[id]) res+=query(rs[id],XL,XR,YL,YR);
        return res;
    }
    int Query(int XL,int XR,int YL,int YR){
        int res=0;
        forup(i,0,17){
            if(root[i]) res+=query(root[i],XL,XR,YL,YR);
        }
        return res;
    }
}mt;
signed main(){
    n=read();
    int lans=0;
    while(1){
        int op=read();
        if(op==1){
            int x=read(),y=read(),a=read();
            x^=lans;y^=lans;a^=lans;
            mt.Update(x,y,a);
        }else if(op==2){
            int xl=read(),yl=read(),xr=read(),yr=read();
            xl^=lans;yl^=lans;xr^=lans;yr^=lans;
            lans=mt.Query(xl,xr,yl,yr);
            printf("%d\n",lans);
        }else{
            break;
        }
    }
}
```

///

////

## 邻域查询

这玩意最坏复杂度还是 $O(n)$ 的，但是很多时候挺快的。

/// admonition | 例题[P2093 [国家集训队] JZPFAR](https://www.luogu.com.cn/problem/P2093)
    type: example

- 平面上有 $n$ 个点，$m$ 次查询，每次查询和一个给定点欧几里得距离第 $k$ 大的点的编号，距离相同则取编号较小者。
- $1\le n\le 10^5,1\le m\le 10^4,1\le k\le 20$，坐标绝对值小于等于 $10^9$。

///

首先我们可以用一个小根堆维护距离前 $k$ 大的点，那么显然若一个子树内最远点都不如前面找出来的第 $k$ 大远，这个子树内就不可能有答案了。

然后若一个点两子树种都有可能有答案，我们可以类似于启发式搜索，写一个估价函数，先进入估价更优的子树。

这道题可以直接用子树内最远距离当作估价函数。

/// details | 参考代码
    open: False
    type: success

```cpp
#include<bits/stdc++.h>
#define forup(i,s,e) for(i64 i=(s),E123123123=(e);i<=E123123123;++i)
#define fordown(i,s,e) for(i64 i=(s),E123123123=(e);i>=E123123123;--i)
#define mem(a,b) memset(a,b,sizeof(a))
#ifdef DEBUG
#define msg(args...) fprintf(stderr,args)
#else
#define msg(...) void()
#endif
using namespace std;
using i64=long long;
using pii=pair<i64,i64>;
#define fi first
#define se second
#define mkp make_pair
#define gc getchar()
i64 read(){
    i64 x=0,f=1;char c;
    while(!isdigit(c=gc)) if(c=='-') f=-1;
    while(isdigit(c)){x=(x<<1)+(x<<3)+(c^48);c=gc;}
    return x*f;
}
#undef gc
const i64 N=1e5+5,inf=1e18;
i64 n,m;
struct Node{
    i64 x,y,v;
};
i64 get(bool ps,Node a){
    return ps?a.y:a.x;
}
i64 dis(i64 x1,i64 y1,i64 x2,i64 y2){
    return (x1-x2)*(x1-x2)+(y1-y2)*(y1-y2);
}
struct KDT{
    i64 ls[N],rs[N],px[N],py[N],pos[N],xl[N],xr[N],yl[N],yr[N],cntn,root;
    void PushUp(i64 id){
        xl[id]=xr[id]=px[id];
        yl[id]=yr[id]=py[id];
        if(ls[id]){
            xl[id]=min(xl[id],xl[ls[id]]);
            xr[id]=max(xr[id],xr[ls[id]]);
            yl[id]=min(yl[id],yl[ls[id]]);
            yr[id]=max(yr[id],yr[ls[id]]);
        }
        if(rs[id]){
            xl[id]=min(xl[id],xl[rs[id]]);
            xr[id]=max(xr[id],xr[rs[id]]);
            yl[id]=min(yl[id],yl[rs[id]]);
            yr[id]=max(yr[id],yr[rs[id]]);
        }
    }
    void Build(i64 &id,bool ps,vector<Node> &vec,i64 l,i64 r){
        if(l>=r) return;
        if(l+1==r){
            id=++cntn;
            px[id]=vec[l].x;py[id]=vec[l].y;
            pos[id]=vec[l].v;
            PushUp(id);
            return;
        }
        i64 mid=(l+r)>>1;
        nth_element(vec.begin()+l,vec.begin()+mid+1,vec.begin()+r,[&](Node a,Node b){return get(ps,a)<get(ps,b);});
        id=++cntn;
        px[id]=vec[mid].x;py[id]=vec[mid].y;
        pos[id]=vec[mid].v;
        Build(ls[id],!ps,vec,l,mid);
        Build(rs[id],!ps,vec,mid+1,r);
        PushUp(id);
    }
    i64 calc(i64 id,i64 X,i64 Y){
        return max({dis(X,Y,xl[id],yl[id]),dis(X,Y,xl[id],yr[id]),dis(X,Y,xr[id],yl[id]),dis(X,Y,xr[id],yr[id])});
    }
    void Query(i64 id,i64 X,i64 Y,priority_queue<pii> &q){
        pii dd=mkp(-dis(X,Y,px[id],py[id]),pos[id]);
        if(dd<q.top()){
            q.pop();q.push(dd);
        }
        i64 vl=ls[id]?calc(ls[id],X,Y):-inf-1,vr=rs[id]?calc(rs[id],X,Y):-inf-1;
        if(vl>=vr){
            if(vl>=-q.top().fi) Query(ls[id],X,Y,q);
            if(vr>=-q.top().fi) Query(rs[id],X,Y,q);
        }else{
            if(vr>=-q.top().fi) Query(rs[id],X,Y,q);
            if(vl>=-q.top().fi) Query(ls[id],X,Y,q);
        }
    }
}mt;
signed main(){
    n=read();
    vector<Node> vec;
    forup(i,1,n){
        i64 x=read(),y=read();
        vec.push_back(Node{x,y,i});
    }
    mt.Build(mt.root,0,vec,0,vec.size());
    m=read();
    forup(i,1,m){
        i64 x=read(),y=read(),k=read();
        priority_queue<pii> q;
        forup(i,1,k) q.push(mkp(inf,inf));
        mt.Query(mt.root,x,y,q);
        printf("%lld\n",q.top().se);
    }
}
```

///

## 其它应用

多维偏序，可以（相比 cdq）更自然地做一些高维偏序下的 DP。

//// details | 例题 [P3769 [CH弱省胡策R2] TATT](https://www.luogu.com.cn/problem/P3769)
    open: False
    type: example

> 题意

- 给定四维超空间中 $n$ 个点，求四维都单调不减的最长点序列的长度。
- $1\le n\le 5\times 10^4$，坐标在 $[-10^9,10^9]$ 之内。

> 题解

排序去掉一维对剩下三维直接 KDT 是 $O(n\log n+n^{\frac{5}{3}})$ 的，不太能过。

考虑一个很妙的算法：树状数组套 KDT。

用树状数组维护其中一维，然后在每个结点上维护 KDT。因为树状数组的结构，在查询时只会访问到每个点一次，复杂度就是单根号的。

于是总复杂度 $O(n\log^2 n+n\sqrt{n})$。

/// details | 参考代码
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
#define msg(...) void()
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
const int N=5e4+5,inf=0x3f3f3f3f;
int n;
struct Point{
    int p[4];
    bool operator <(const Point &r)const{
        if(p[0]!=r.p[0]) return p[0]<r.p[0];
        if(p[1]!=r.p[1]) return p[1]<r.p[1];
        if(p[2]!=r.p[2]) return p[2]<r.p[2];
        return p[3]<r.p[3];
    }
}s[N];
struct Node{
    int x,y,po;
};
int get(bool ps,Node a){
    if(ps) return a.y;
    else   return a.x;
}
struct KDT{
    int ls[N*20],rs[N*20],cntn;
    int mx[N*20],ff[N*20],val[N*20];
    int px[N*20],py[N*20],xl[N*20],xr[N*20],yl[N*20],yr[N*20];
    void PushUp(int id){
        mx[id]=val[id];
        xl[id]=xr[id]=px[id];
        yl[id]=yr[id]=py[id];
        if(ls[id]){
            xl[id]=min(xl[id],xl[ls[id]]);
            xr[id]=max(xr[id],xr[ls[id]]);
            yl[id]=min(yl[id],yl[ls[id]]);
            yr[id]=max(yr[id],yr[ls[id]]);
            mx[id]=max(mx[id],mx[ls[id]]);
            ff[ls[id]]=id;
        }
        if(rs[id]){
            xl[id]=min(xl[id],xl[rs[id]]);
            xr[id]=max(xr[id],xr[rs[id]]);
            yl[id]=min(yl[id],yl[rs[id]]);
            yr[id]=max(yr[id],yr[rs[id]]);
            mx[id]=max(mx[id],mx[rs[id]]);
            ff[rs[id]]=id;
        }
    }
    void Build(int &id,bool ps,vector<Node> &vec,map<int,int> &mp,int l,int r){
        if(l>=r) return;
        if(l+1==r){
            id=++cntn;
            px[id]=vec[l].x;py[id]=vec[l].y;
            mp[vec[l].po]=id;
            PushUp(id);
            return;
        }
        int mid=(l+r)>>1;
        nth_element(vec.begin()+l,vec.begin()+mid+1,vec.begin()+r,[&](Node a,Node b){
            return get(ps,a)<get(ps,b);
        });
        id=++cntn;
        px[id]=vec[mid].x;py[id]=vec[mid].y;
        mp[vec[mid].po]=id;
        Build(ls[id],!ps,vec,mp,l,mid);
        Build(rs[id],!ps,vec,mp,mid+1,r);
        PushUp(id);
    }
    int Query(int id,int XL,int XR,int YL,int YR){
        if(XL<=xl[id]&&xr[id]<=XR&&YL<=yl[id]&&yr[id]<=YR) return mx[id];
        if(XR<xl[id]||XL>xr[id]||YR<yl[id]||YL>yr[id]) return 0;
        int res=0;
        if(XL<=px[id]&&px[id]<=XR&&YL<=py[id]&&py[id]<=YR) res=max(res,val[id]);
        if(ls[id]) res=max(res,Query(ls[id],XL,XR,YL,YR));
        if(rs[id]) res=max(res,Query(rs[id],XL,XR,YL,YR));
        return res;
    }
    void Update(int id,int v){
        val[id]=v;
        while(id){
            PushUp(id);
            id=ff[id];
        }
    }
}t1;
int sz;
struct BIT{
    vector<Node> vec[N];
    map<int,int> mp[N];
    void UpdateP(int x,Node k){
        for(;x<=sz;x+=x&-x){
            vec[x].push_back(k);
        }
    }
    int root[N];
    void Build(){
        forup(i,1,sz){
            t1.Build(root[i],0,vec[i],mp[i],0,vec[i].size());
        }
    }
    void UpdateV(int x,int p,int v){
        for(;x<=sz;x+=x&-x){
            t1.Update(mp[x][p],v);
        }
    }
    int Query(int x,int XR,int YR){
        int res=0;
        for(;x>0;x-=x&-x){
            res=max(res,t1.Query(root[x],-inf,XR,-inf,YR));
        }
        return res;
    }
}t2;
signed main(){
    n=read();
    vector<int> lsh;
    forup(i,1,n){
        forup(j,0,3){
            s[i].p[j]=read();
        }
        lsh.push_back(s[i].p[1]);
    }
    sort(s+1,s+n+1);
    sort(lsh.begin(),lsh.end());
    lsh.erase(unique(lsh.begin(),lsh.end()),lsh.end());
    sz=lsh.size();
    forup(i,1,n){
        s[i].p[1]=lower_bound(lsh.begin(),lsh.end(),s[i].p[1])-lsh.begin()+1;
    }
    forup(i,1,n){
        t2.UpdateP(s[i].p[1],Node{s[i].p[2],s[i].p[3],i});
    }
    t2.Build();
    int ans=0;
    forup(i,1,n){
        int val=t2.Query(s[i].p[1],s[i].p[2],s[i].p[3])+1;
        t2.UpdateV(s[i].p[1],i,val);
        ans=max(ans,val);
    }
    printf("%d\n",ans);
}
```

///

////