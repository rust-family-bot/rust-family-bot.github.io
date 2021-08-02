---
layout: post
title:  "roblas 开发(1) - 项目组织"
date:   2021-06-27 16:27:00 +0800
categories: [roblas]
tags: [roblas, rust]
---

# roblas 开发(1) - 项目组织

此章节主要描述 roblas 项目目前的组织。

在目前的想法中，roblas 的总体项目目录为：

```
.
├── Cargo.toml
├── LICENSE
├── README.md
├── doc			// 用于 cargo doc 代码生成
├── src
│   ├── common		// 共享的类型定义和常量等
│   ├── error.rs	// 错误处理相关
│   ├── level1		// level 1 实现
│   │   ├── mod.rs
│   │   └── naive	// 无平台相关优化版本
│   ├── level2		// level 2 实现
│   │   ├── mod.rs
│   │   └── naive	// 无平台相关优化版本
│   ├── level3		// level 3 实现
│   │   ├── mod.rs
│   │   └── naive	// 无平台相关优化版本
│   ├── lib.rs		// lib 主入口
│   └── utils.rs	// 通用辅助函数
└── tests
    ├── level1		// level 1 测试
    │   ├── c_test.rs	// c 类（单精度复数）函数测试
    │   ├── d_test.rs	// d 类（双精度复数）函数测试
    │   ├── mod.rs
    │   ├── s_test.rs	// s 类（单精度）函数测试
    │   └── z_test.rs	// z 类（双精度）函数测试
    ├── level2		// level 2 测试
    ├── level2		// level 3 测试
    └── test.rs		// 测试主入口
```

对于每一个 level 的函数，都会对应一个 module。在每一个 level 的 module 中，至少含有一个 naive 的实现（naive 实现即不含有任何平台相关优化）。在之后的版本中，将会在 module 中加入其他平台相关的优化，并通过条件编译控制加入。

