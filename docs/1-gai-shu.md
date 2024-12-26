---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 1 概述

> 本书中所说的`yarn`统一指`yarn1`，如果是`yarn2`或者`yarn3` `yarn4` 会特别的著名版本号。

### yarn的简介

`yarn`是一个老版的包管理工具，并一直处于迭代阶段。在2020年之后很多新型的包管理工具，比如`pnpm` `yarn2`(也就是之后的berry) 都发展起来，但yarn现在(2024)也是一个很强大的包管理工具，很多新型的包管理工具比如`pnpm`都沿用了`yarn`里面的设计和概念。比较知名的是`resolution` 以及`patch`。这些概念知道现在还被各个包管理工具沿用。由此可见`yarn`是一个很强大的工具。

理解`yarn`的原理有助于理解各个包管理工具的基本概念以及熟练使用各种包管理工具。对于`yarn`原理的理解，本书用解析源码的方式来展示`yarn`是如何工作并运行的。另外本书会提及较少新的包管理工具如`pnpm`以及`yarn4`的知识。

### 为什么选择yarn

包管理工具有很多本书为什么偏偏选`yarn`来讲。

对于新型的包管理工具来讲，比如`pnpm`以及`yarn4`，整个源代码体积太大内容太多，组织源代码的形式都是`monorepo`，很多关键的信息不明显。

对比`npm`来讲，`npm`很多关系的代码都是依赖的外部的包，源码并没有在同一个仓库里面，调试起来也很麻烦。

`yarn`相比之下内容都在一个仓库里面[github/yarn/yarn](https://github.com/yarnpkg/yarn)，而且`yarn`最近4年基本无新的commit，很早就不再继续迭代而且在`berry`上进行迭代，十分有利于调试。