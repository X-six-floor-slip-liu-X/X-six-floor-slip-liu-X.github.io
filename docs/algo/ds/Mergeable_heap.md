---
comments: true
---

# 可并堆学习笔记

## 前言

没啥说的，很简单的一类数据结构，就是可以合并的堆。

一般来说合并两个堆需要一边慢慢弹另一边慢慢压，复杂度 $O(n\log n)$，但是可并堆可以做到单 $\log$（貌似还有更好的复杂度，但是我不会）。

本文中均以大根堆为例。

## 斜堆

最好理解的可并堆。

考虑暴力合并，类似于 FHQ 的合并操作。具体来说，将两个堆的根相比较，选较大的那个作为新的根，然后将它的某个儿子与另一个堆递归求解。

但是假如你一直选右儿子（或者一直选左儿子），树高就会被卡到 $O(n)$，怎么办呢？一个简单的想法是随机选左右儿子（其实也可以每次有一半概率交换左右儿子），然后就不容易被卡了。

/// details | 参考代码
    open: False
    type: success

```cpp
const int N=1e5+5,inf=0x3f3f3f3f;
int n,m,a[N];
int fa[N],vis[N];
int getfa(int x){return x==fa[x]?x:fa[x]=getfa(fa[x]);}
int root[N],son[N][2];
mt19937 mr(time(0));
bool Rd(){
	return (unsigned int)mr()%2;
}
int Merge(int u,int v){
	if(!u||!v) return u|v;
	if(a[u]<a[v]||(a[u]==a[v]&&u<v)){
		bool ss=Rd();
		son[u][ss]=Merge(son[u][ss],v);
		return u;
	}else{
		bool ss=Rd();
		son[v][ss]=Merge(son[v][ss],u);
		return v;
	}
}
int pop(int u){
	if(!son[u][0]&&!son[u][1]){
		return 0;
	}else if(son[u][0]&&son[u][1]){
		if((a[son[u][0]]<a[son[u][1]])||(a[son[u][0]]==a[son[u][1]]&&son[u][0]<son[u][1])){
			son[son[u][0]][0]=pop(son[u][0]);
			son[son[u][0]][1]=son[u][1];
			return son[u][0];
		}else{
			son[son[u][1]][1]=pop(son[u][1]);
			son[son[u][1]][0]=son[u][0];
			return son[u][1];
		}
	}else{
		return son[u][0]|son[u][1];
	}
}
signed main(){
	n=read();m=read();
	forup(i,1,n){
		a[i]=read();
		root[i]=i;
		fa[i]=i;
	}
	forup(i,1,m){
		int op=read();
		if(op==1){
			int x=read(),y=read(),fx=getfa(x),fy=getfa(y);
			if(vis[x]||vis[y]||fx==fy){
				continue;
			}
			fa[fx]=fy;
			root[fy]=Merge(root[fx],root[fy]);
		}else{
			int x=read(),fx=getfa(x);
			if(vis[x]){
				puts("-1");
				continue;
			}
			printf("%d\n",a[root[fx]]);
			vis[root[fx]]=1;
			root[fx]=pop(root[fx]);
		}
	}
}
```

///

## 左偏树

如果你不喜欢随机化算法，还有复杂度严格的左偏树。

首先考虑堆的左右儿子是等价的（显然吧，因为堆只有儿子和父亲之间有关系）。那么我们可以在任何时候交换左右儿子。

观察到合并的复杂度只和我们走的那一条链的长度有关，容易想到只要让某一条链小于等于其它任意链，这条链的长度显然就是严格 $O(\log n)$ 的。

所以我们在合并的时候再额外记一个**和最近的叶子结点的距离**，然后每次回溯的时候保证右边的距离小于等于左边的（大了就交换一下），合并的时候一直走右儿子即可。

但是左偏树有个弊端，就是左边的路径可能被卡到 $O(n)$，在某些题可能会有形如“从 $x$ 暴力往上跳”的操作，复杂度就假了。斜堆就没有这个问题。

/// detials | 参考代码
    open: False
    type: success

其余部分一模一样。

```cpp
int Merge(int u,int v){
	if(!u||!v) return u|v;
	if(a[u]<a[v]||(a[u]==a[v]&&u<v)){
		son[u][1]=Merge(son[u][1],v);
		dis[u]=min(dis[son[u][0]],dis[son[u][1]])+1;
		if(dis[son[u][0]]<dis[son[u][1]]) swap(son[u][0],son[u][1]);
		return u;
	}else{
		son[v][1]=Merge(son[v][1],u);
		dis[v]=min(dis[son[v][0]],dis[son[v][1]])+1;
		if(dis[son[v][0]]<dis[son[v][1]]) swap(son[v][0],son[v][1]);
		return v;
	}
}
```

///