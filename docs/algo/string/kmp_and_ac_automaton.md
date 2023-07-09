---
comments: true
---

# KMP 与 AC 自动机

## 前言

这篇文章是我学习 AC 自动机的学习笔记，但是因为我以前对 KMP 也不了解在学 AC 自动机时就重新复习了一下。

## 前缀函数与 kmp 算法

在正文之前，先复习一下字符串中的一些基础定义：

/// admonition | 字符串基础定义
    type: info
- 前缀

**前缀**是一个字符串从串首到某个位置结束的特殊子串。其中**真前缀**表示除了该字符串本身以外的所有前缀。

- 后缀

**后缀**与**真后缀**的定义和前缀类似。

///