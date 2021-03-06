---
layout: post
title:  "roblas 开发(2) - blas 矩阵格式说明"
date:   2021-07-21 19:43:00 +0800
categories: [roblas]
tags: [roblas, rust]
---

# roblas 开发(2) - blas 矩阵格式说明

此章节主要描述 roblas 项目中 level 2 及 level 3 中涉及到的矩阵输入格式说明。

本章节主要参考 [https://zhuanlan.zhihu.com/p/104287878](https://zhuanlan.zhihu.com/p/104287878)。

## 常量

首先，在开始之前，需要先关注 `src/common/constant.rs` 文件中所定义的几种 enum 类型，虽然其使用 rust 的枚举类型表示，实际上使用 `i32` 的数据类型对 enum 中的数据进行存储（`#[repr(i32)]`。此外，为方便报错信息的打印，也同时实现了 `#[derive(Debug)]`。

enum 的数值含义，与其名称相符合，自行意会。

## fortran 的 blas 中的矩阵表示

fortran 很怪，真的。

现在而言，通常情况下，我们表达一个矩阵，都是使用二维数组的，例如下面的矩阵 $\boldsymbol{A}$

$$
\boldsymbol{A}=\left [ \begin{matrix}
1 & 2 \\
3 & 4 \\
5 & 6
\end{matrix} \right ]
$$

我们可以设置一个二维数组 $a[i][j]$，其中有 

$$
a[0][0]=1,a[0][1]=2 \\
a[1][0]=3,a[1][1]=4 \\
a[2][0]=5,a[2][1]=6
$$

这是一个非常朴素的想法，而且没有任何谬误。

不过二维数组的概念在 fortran77 那个年代或许有点超前，又或者不太实用，所以 fortran 中选择将二维数组压缩为一维数组。在这个压缩的过程中，势必会面临“**列优先**”或者“**行优先**”的问题。

简单地说，对于一个矩阵或者二维数组，有两种等价的表达方式。继续以上述的矩阵 $\boldsymbol{A}$ 作为例子，使用一维数组 $b$ 表示，则其**行优先**表示为

$$
b[0]=1,b[1]=2,b[2]=3,b[3]=4,b[4]=5,b[5]=6 \\
m=3 \\
n=2
$$

其中矩阵可以表示为 $m\times n$ 的形式。

事实上，在现在的 c 语言等高级编程语言中，对于上面的二维数组 $a$ 而言，可以直接当作一维数组 $b$ 使用，因为大多数语言中的多维数组，都是使用**行优先**的表示方式进行存储的。

虽然但是，fortran blas 偏偏不喜欢这一套，因为他是采用**列优先**的方式存储的。也就是说，如果使用一维数组 $c$ 表示同一个矩阵 $\boldsymbol{A}$，结果为：

$$
c[0]=1,c[1]=3,c[2]=5,c[3]=2,c[4]=4,c[5]=6 \\
m=3 \\
n=2
$$

矩阵仍然是 $m\times n$，但是存储的形式为**列优先**。

理解到这一层后，对于 blas 的 fortran 实现中的矩阵形态就没有迷惑了。

<span id="simple"></span>

### 一个简单结论

值得一提的是，我们可以发现，如果我们对矩阵转置得到 $\boldsymbol{A}^T$，则其原来的**列优先**存储形式的一维数组 $c$，就和转置后的**行优先**存储形式的一维数组 $b^T$ 相同。（十分显然的结论）

这个结论的应用比较常见，例如对于 level 2 的 gemv 系列函数，其作用为计算 

$$
y\gets \alpha*op(\boldsymbol{A})*\vec{x}+\beta*\vec{y}
$$

其中 $\boldsymbol{A}$ 在传入函数时，我们一定认为其为 $m\times n$ 的**列优先**矩阵。$op(\boldsymbol{A})$ 可为 $\boldsymbol{A},\boldsymbol{A}^T$ 两者中的一种（可以通过参数控制），在这里我们认为 $op(\boldsymbol{A})=\boldsymbol{A}$，即此时，我们正在计算

$$
y\gets \alpha*\boldsymbol{A}*\vec{x}+\beta*\vec{y}
$$

虽然但是，如果好死不死，我们传入的矩阵 $\boldsymbol{A}$ 为**行优先**矩阵，那么我们可以通过调控参数，使

$$
op(\boldsymbol{A})=\boldsymbol{A}^T
$$

则我们也可以得到正确的结果。（动动手啊动动脑，想想这是为什么）

## fortran blas 中的 lda 参数

光是理解矩阵存储形式还是不足够理解 blas 函数中的功能，剩下一个关键点——`lda` 参数的含义仍然不明晰。下文仍然使用上面的**列优先**矩阵 $\boldsymbol{A}$ 作为例子，大小为 $m\times n=3\times 2$，对应的一维数组为 $c$。

计算的过程中，我们可能会使用矩阵的分块乘法的技巧来加速矩阵运算，因此我们需要能够在一个较大的矩阵 $\boldsymbol{A}$ 中，表示其中的一个小部分 $\boldsymbol{A}'$。

此处，我们假设选取原来的矩阵的上部的 $m'\times n=2\times 2$ 部分，即

$$
\boldsymbol{A}'=\left [ \begin{matrix}
1 & 2 \\
3 & 4
\end{matrix} \right ]
$$

此时可以发现，矩阵 $\boldsymbol{A}'$ 的一维数组 $c'$ 的内容与原来的数组 $c$ 不同，而且维度也不一样，所以我们自然想到需要创建一个新的数组，并对应的内容于 $c'$ 中。

这个想法非常朴素且可行，只可惜这样子内存消耗太大，如果矩阵大一点，分块次数多一些，迟早内存吃光光！因此，我们换一种思路——直接共用同一个一维数组指针 $c$。

但是直接使用 $c$ 是不行的，这个时候就轮到 `lda` 出场力。lda，全称 leading dimension of array，就是用来解决子矩阵的问题。`lda` 的值大小，等于**当前 X 优先的表示形式中，要使有效数据的第一个维度增加 1，所需要增加的大小**。

回忆起原来的矩阵为

$$
\boldsymbol{A}=\left [ \begin{matrix}
1 & 2 \\
3 & 4 \\
5 & 6
\end{matrix} \right ]
$$

注意到 $c$ 采用**列优先**存储方式，如果将其直接当作二维数组，则其第一个维度即为**列**，第二个维度为**行**，因此 `lda` 的值为**在 $\boldsymbol{A}'$ 中，要使列维度增加 1，在 $c$ 中所需要增加的大小**。

显然，在**列优先**形式下，此时 `lda` 的大小为**原矩阵 $\boldsymbol{A}$ 的行数**。

直观的理解下，已知 `lda` 为 3，则我们对矩阵 $\boldsymbol{A}$ 读取完左上的 1 和 3 之后，因为 $\boldsymbol{A}'$ 的大小为 $2\times 2$，所以我们需要读取下一列，因此我们根据 `lda` 的值，知道从这一列的开始元素 1 到下一列的开始元素之间，需要 3 个元素的地址偏移，之后就能得到下一列的开始元素为 2。

小朋友，明白了吗？

## cblas 对 fortran blas 的封装

如果有好好理解上面的内容，对于 cblas 的封装，应该就会感觉很简单。

相对于 fortran 默认传入的矩阵的一维数组为**列优先**，在 cblas 中，允许用户指定传入的一维数组的存储形式。

事实上，cblas 只是使用上面的[一个简单的结论](#simple)一章中的内容，在 c 语言的层面，通过操控 $op(\boldsymbol{A})$，从而改变存储形式，然后得到正确的结果罢了，因此这里没什么好说的。

好了，该开工了小朋友。
