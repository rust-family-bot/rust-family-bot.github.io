---
layout: post
title:  "roblas 上游 - blas 库"
date:   2021-06-27 04:34:00 +0800
categories: [roblas]
tags: [roblas, rust]
---

# roblas 上游 - blas 库

## BLAS 基本说明

此处基本参考：[https://blog.csdn.net/hehe199807/article/details/108365427](https://blog.csdn.net/hehe199807/article/details/108365427)

BLAS 库，即 Basic Linear Algrbra Subprograms，是进行向量和矩阵等基本线性代数操作的事实上的数值库。这些程序最早在1979年发布，是LAPACK(Linear Algebra PACKage)的一部分，便于建立功能更强的数值程序包。BLAS库在高性能计算中被广泛应用，由此衍生出大量优化版本，如Intel MKL，AMD 的 ACML，ATLAS 等。在 cuda 计算领域，也有实现了相应接口的 cublas 库存在。

### BLAS 构成

* BLAS1：支持向量与向量之间相关的操作。主要是为了编程方便，在实际计算中使用很少，因为不能在大多数机器上实现高性能。
* BLAS2：支持矩阵与向量之间相关的操作。在 $O(n^2)$ 时间复杂度实现 $O(n^2)$ 的浮点运算。在许多向量机上能实现接近峰值的性能。然而在多级存储器的处理器上则性能一般，因为受限于数据在存储器间的迁移。
* BLAS3：支持矩阵与矩阵之间相关的操作。在 $O(n^2)$ 时间复杂度实现 $O(n^3)$ 量级的浮点运算，能充分发挥现代处理器的性能，并且能为用户提供透明的并发机制。

### BLAS 函数介绍

BLAS 中对每个函数的功能定义见官方：[http://netlib.org/blas/blasqr.pdf](http://netlib.org/blas/blasqr.pdf)。BLAS 函数名中的前缀 “`x`” 表示参数和返回值的类型：

* S：单精度实数
* D：双精度实数
* C：单精度复数
* Z：双精度复数
* Q：混合精度

其中混合精度由 S，D，C，Z 组成，最左边表示返回值类型，其余为参数类型。如 DROTMG 表示的是双精度的平面转换，SSCAL 表示的则是单精度的向量缩放。

### BLAS 主要参数

此处有一个解释较好的回答：[https://zhuanlan.zhihu.com/p/104287878](https://zhuanlan.zhihu.com/p/104287878)。

在 BLAS2 和 BLAS3 的函数中，有以下重要的参数：

* INCX： 向量X的存储步长
* LDA： 矩阵A的存储步长(主维(Leading Dimension)实际占用的空间)。这个参数查看上述的知乎链接，若无法理解，可以观看老外视频：[https://www.youtube.com/watch?v=PhjildK5oO8](https://www.youtube.com/watch?v=PhjildK5oO8)
* TRANS： 矩阵的转置形式，T(实数转置)、N(实数非转置)、C(复数共轭转置)、H(复数共轭非转置)
* UPLO： 对称阵或三角阵的存储方式，U(上三角)、L(下三角)
* DIAG： 对角线形式，N(非单位对角线)、U(单位对角线)
* SIDE： 矩阵在操作中的位置，L(左边)、R(右边)



### CBLAS 接口与 OpenBLAS

正如之前所说，BLAS 只是一组向量和矩阵运行的接口规范，也可以称为API规范。Netlib 用 fortran 实现了 BLAS 的这些 API 接口，得到的库也叫做 BLAS。Netlib只是一般性地实现了基本功能，并没有对运算做过多的优化。

LAPACK （linear algebra package），是著名的线性代数库，也是 Netlib 用 fortran 语言编写的。其底层是 BLAS，在此基础上定义了很多矩阵和向量高级运算的函数，如矩阵分解、求逆和求奇异值等。该库的运行效率比 BLAS 库高。
从某个角度讲，LAPACK 也可以称作是一组科学计算（矩阵运算）的接口规范。Netlib 实现了这一组规范的功能，得到的这个库叫做 LAPACK 库。
前面 BLAS 和 LAPACK 的实现均是用的 Fortran 语言。为了方便 c 程序的调用，Netlib 开发了 CBLAS 和 CLAPACK。其本质是在 BLAS 和 LAPACK 的基础上，增加了 c 的调用方式。

再有了接口规范之后，其他的组织、个人和公司，就可以根据此规范，实现自己的科学计算库。
开源社区实现的科学计算（矩阵计算）库中，比较著名的两个就是 atlas 和 OpenBLAS。它们都实现了 BLAS 的全部功能，以及 LAPACK 的部分功能，并且他们都对计算过程进行了优化。



下面是一些相关链接：

* netlib 实现的 blas 库：http://netlib.org/blas/
* atlas 库：[https://www.netlib.org/atlas/](https://www.netlib.org/atlas/)
* lapack 库：[http://www.netlib.org/lapack/](http://www.netlib.org/lapack/)



## roblas 实现参考

目前的计划，是依照 netlib 的 cblas 库进行初版 roblas 的实现，即函数名需要遵守 c 规范，可能考虑后续与 c 进行链接测试。

平台相关的优化，预计在明年再实现。

