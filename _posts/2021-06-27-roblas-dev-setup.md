---
layout: post
title:  "roblas 开发准备"
date:   2021-06-27 02:56:53 +0800
categories: [roblas]
tags: [roblas, rust]
---

# roblas 开发准备

这一章节中，将会涉及到 roblas 开发之前的环境准备。

## rust

### 安装

安装 rust 的操作详见 rust 官方：[https://www.rust-lang.org/](https://www.rust-lang.org/)

### 语言准备

学习 rust 语言，建议直接从官方的英文版 rust book 开始：[https://doc.rust-lang.org/book/](https://doc.rust-lang.org/book/)。

除此以外，有时间也可阅读 rust by example 一书：[https://doc.rust-lang.org/stable/rust-by-example/](https://doc.rust-lang.org/stable/rust-by-example/)。

最后，作为阶段性检查，建议完成官方强推的 rustlings 练习：[https://github.com/rust-lang/rustlings](https://github.com/rust-lang/rustlings)。



## roblas

### roblas 获取

输入命令：

```sh
git clone git@github.com:rust-family/roblas.git
```

### roblas 运行

目前的 roblas 拥有最简单的框架，可以直接构建：

```sh
cargo build
```

roblas 也同时附有测试框架：

```sh
cargo test
```

得益于 rust 的 doc 机制，输入下面命令，可以生成 doc 网页：

```sh
RUSTDOCFLAGS="--html-in-header doc/math-header.html" cargo doc --no-deps --open
```

目前的 doc 生成过程中，通过引入 `doc/math-header.html` ，使得 docstring 中可以加入 markdown 格式的数学公式，例如 $\hat{x} \to \alpha \cdot \hat{x}$。



## Apache2 License

目前选用 Apache2 License，理论上每个文件头都需要放入声明，早期可以忽略，再最后发版前再补齐亦可。



## Github workflow

目前在项目的 `main` 和 `dev` 分支中架设了 Github Action 的 workflow，即 CI/CD 服务。在项目根目录下的 `.github/workflow` 目录中，存放着不同分支的 CI/CD 规则。

当前的规则为分支检测到 push 或者 pull request 时，进行一次 `cargo build` 并 `cargo test`，以保证 commit 的正确性。

在项目的 `README.md` 的开头也放入了相应的 badge。



## 开发分支管理

为了更好的管理 Git 分支，我们会有 main 以及 dev 分支。

main 分支用来发布主要的版本。

dev 分支用于同步主要的开发进展。

个人开发时，需要创建新分支，分支名的格式如 `[开发者]/[分支功能]`，例如 `ayajilin/algorithm` 表示为开发者 ayajilin 开发算法的分支。个人开发分支结束后，需要通过 **rebase** 先 rebase 到 dev 分支，再进行合并提交。

