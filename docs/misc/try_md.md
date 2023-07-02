# 常用 MarkDown 语法

众所周知 MarkDown 是一种轻量级的标记语言，这里简单归纳一下常用语法（其实是想测试一下这个博客的渲染功能有没有问题）。

## 代码块
```cpp
#include<iostream>
int main(){
    int a,b;cin>>a>>b;
    cout<<a+b;
    return 0;
}
```

````
```cpp
#include<iostream>
int main(){
    int a,b;cin>>a>>b;
    cout<<a+b;
    return 0;
}
```
````

如何在代码块内套代码块：

`````
````
```cpp
#include<iostream>
int main(){
    int a,b;cin>>a>>b;
    cout<<a+b;
    return 0;
}
```
````
`````
再往外一层也同理。

### 行内代码块

这道题需要开 `long long`。

```
这道题需要开 `long long`。
```
## 标题

```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

## 列表

### 有序列表

1. 第一个问题
1. 第二个问题
1. 第三个问题

```
1. 第一个问题
1. 第二个问题
1. 第三个问题
```

### 无序列表

- 问题 A
- 问题 B
- 问题 C

```
- 问题 A
- 问题 B
- 问题 C
```

## 特殊字体

**黑体** __黑体__

~~删除线~~

*斜体* _斜体_

```
**黑体** __黑体__

~~删除线~~

*斜体* _斜体_
```

## 公式

$\sum_{i=1}^{n}i=\frac{n(n+1)}{2}$

```
$\sum_{i=1}^{n}i=\frac{n(n+1)}{2}$
```
支持换行，但不知道为什么要打四个斜杠。

$$
\begin{aligned}
  &ac+ad+bc+bd\\\\
= &a(c+d)+b(c+d)\\\\
= &(a+b)(c+d)
\end{aligned}
$$

```
$$
\begin{aligned}
  &ac+ad+bc+bd\\\\
= &a(c+d)+b(c+d)\\\\
= &(a+b)(c+d)
\end{aligned}
$$
```

## Mkdocs 拓展 MarkDown 语法

### PyMdown blocks

!!! note "这是 note 类型的提示框"
    提示

??? note "这是 note 类型的折叠的提示框"
    提示

!!! success "这是 success 类型的提示框"
    成功！

!!! failure "这是 failure 类型的提示框"
    失败！

!!! bug "这是 bug 类型的提示框"
    发现一个 bug，请尽快修复！

```
!!! note "这是 note 类型的提示框"
    提示

??? note "这是 note 类型的折叠的提示框"
    提示

!!! success "这是 success 类型的提示框"
    成功！

!!! failure "这是 failure 类型的提示框"
    失败！

!!! bug "这是 bug 类型的提示框"
    发现一个 bug，请尽快修复！
```

/// admonition | 新写法
    type: note
这是一种不用缩进的写法。
///

/// tab | Tab 1
相邻的 tab 会并排显示。
///

/// tab | Tab 2
这里 Tab 1 和 Tab 2 就并排显示了。
///

```
/// admonition | 新写法
    type: note
这是一种不用缩进的写法。
///

/// tab | Tab 1
相邻的 tab 会并排显示。
///

/// tab | Tab 2
这里 Tab 1 和 Tab 2 就并排显示了。
///
```

/// details | 折叠的提示框
    open: False
    type: note
这是一个默认折叠的 note 类型的提示框
///

/// details | 折叠的提示框
    open: True
    type: success
这是一个默认开启的 success 类型的提示框
///

```
/// details | 折叠的提示框
    open: False
    type: note
这是一个默认折叠的 note 类型的提示框
///

/// details | 折叠的提示框
    open: True
    type: success
这是一个默认开启的 success 类型的提示框
///
```
### 代码块拓展

```cpp title='A+B' linenums="1" hl_lines="3 4"
#include<iostream>
int main(){
    int a,b;cin>>a>>b;// (1)
    cout<<a+b;// (2)!
    return 0;
}
```

1.  用 `cin` 读入。
2.  用 `cout` 输出。

````
```cpp title='A+B' linenums="1" hl_lines="3 4"
#include<iostream>
int main(){
    int a,b;cin>>a>>b;// (1)
    cout<<a+b;// (2)!
    return 0;
}
```

1.  用 `cin` 读入。
2.  用 `cout` 输出。
````

这里有**代码块标题**，**显示行号**，**代码高亮**，**自定义注释（注意自定义注释与代码块本体之间空一行）**。其中自定义注释后面加个 `!` 就可以隐藏原本的斜线了。