# 1 yarn概述

> 本文发布于[掘金专栏](https://juejin.cn/column/7452635467849105459)

> 文章原文位于[github](https://github.com/2239559319/yarn-principle-analysis)

> 本书中所说的`yarn`统一指`yarn1`，如果是`yarn2`或者`yarn3` `yarn4` 会特别的著名版本号。

## yarn的简介

`yarn`是一个老版的包管理工具，并一直处于迭代阶段。在2020年之后很多新型的包管理工具，比如`pnpm` `yarn2`(也就是之后的berry) 都发展起来，但yarn现在(2024)也是一个很强大的包管理工具，很多新型的包管理工具比如`pnpm`都沿用了`yarn`里面的设计和概念。比较知名的是`resolution` 以及`patch`。这些概念知道现在还被各个包管理工具沿用。由此可见`yarn`是一个很强大的工具。

理解`yarn`的原理有助于理解各个包管理工具的基本概念以及熟练使用各种包管理工具。对于`yarn`原理的理解，本书用解析源码的方式来展示`yarn`是如何工作并运行的。另外本书会提及较少新的包管理工具如`pnpm`以及`yarn4`的知识。

## 为什么选择yarn

包管理工具有很多本书为什么偏偏选`yarn`来讲。

对于新型的包管理工具来讲，比如`pnpm`以及`yarn4`，整个源代码体积太大内容太多，组织源代码的形式都是`monorepo`，很多关键的信息不明显。

对比`npm`来讲，`npm`很多关系的代码都是依赖的外部的包，源码并没有在同一个仓库里面，调试起来也很麻烦。

`yarn`相比之下内容都在一个仓库里面[github/yarn/yarn](https://github.com/yarnpkg/yarn)，而且`yarn`最近4年基本无新的commit，很早就不再继续迭代而且在`berry`上进行迭代，十分有利于调试。

## yarn的发展历史

从[yarn官网blog](https://classic.yarnpkg.com/lang/en/)来看`yarn`最开始于2016年发布，最开始的主打的就是一个快。一直保持很频繁的迭代知道2020年。2020年开始`yarn`团队主要投入到当时`yarn2`的开发，也就是`berry`版本，也就是现在的最新版`yarn4`的前身。从[github/yarn仓库](https://github.com/yarnpkg/yarn)来看，`yarn`从2020年开始就不再迭代，只是提交了几个细小的配置问题，剩余的`pr`全部没有合入。现在所有的feature以及bug都在`berry`的[仓库](https://github.com/yarnpkg/berry)进行修改

截至2024.12.26，`yarn`的最新版本是`1.22.22`

## 源码调试

接下来是调试源码的操作指南

### 环境准备

需要准备的环境有`node`，`yarn`，以及`git`

运行以下命令

```bash
git clone https://github.com/yarnpkg/yarn.git

cd yarn

git checkout -b mybuild-v1.22.22

git reset --hard v1.22.22

yarn
```

由于`yarn`源码的包管理工具还是`yarn`，所以准备的环境中需要有`yarn`。很多的包管理工具源码的包管理工具也是自己，比如`pnpm`的源码是用`pnpm`来管理，`npm`的源码也是使用`npm`自己来管理

### 代码分析

通过`package.json`看到`yarn`的源码使用`flow`来编写，使用`babel`以及`flow 的babel插件`来完成编译。为了统一流程`yarn`使用了`gulp`。常用的编译是`build script`，这是把编译到`lib`目录下，这种编译并不打包。还有一种是`build-bundle`，这是使用`webpack`把所有`yarn`的代码以及依赖的代码全部打到一个文件里面，`corepack`使用的`yarn`就是使用的这种形式的打包。

`yarn`有一些依赖比如`package.json`里面声明的`debug`以及`commander`，这些依赖在全部安装`yarn`时会一起安装到全局的`node_modules`文件夹里面。在`build-bundle`模式下则和`yarn`的源码被一起打包到了一个单独的文件。

yarn的依赖

![yarn的依赖](https://unpkg.com/xiaochuan-static-dev@0.0.1/dist/bd002718baebd9a2.png)

`yarn`打包使用的`babel`在现在已经时一个很古老的版本了，`yarn`使用的`babel6`，现在主流时`babel7`而且两个版本的插件不兼容，包名也不一样。

为了调试需要编译出带有正确`sourcemap`的产物文件。这里直接使用新的`babel`版本以及`@babel/preset-flow`来编译。由于`yarn`的源码中大部分的语法都是`es6`的语法，所以不需要`@babel/preset-env`，直接生成`es6`语法的代码。同时为了编译出的`module`是`commonjs`需要使用`@babel/plugin-transform-modules-commonjs`。

### 打包调试版本

通过上面的分析安装`babel`有关的包

```bash
yarn add -D @babel/core @babel/cli @babel/preset-flow @babel/plugin-transform-modules-commonjs
```

新建`babel.config.js`配置文件

```js
module.exports = {
  presets: ['@babel/flow'],
  plugins: [
    [
      '@babel/plugin-transform-modules-commonjs',
      {
        lazy: () => true,
      },
    ],
  ],
};
```

写入打包的script

```json
{
  "mybuild": "babel --no-babelrc src -s -d mybuild"
}
```

运行`yarn mybuild`或者直接运行`yarn babel --no-babelrc src -s -d mybuil`。可以看到`mybuild`目录生成了带有`sourcemap`的产物文件。

运行`node mybuild/cli/index.js --version`输出`1.22.22`

![yarn --version输出](https://unpkg.com/xiaochuan-static-dev@0.0.1/dist/806cfe4b473301ba.png)

断点进`src/cli/index.js`的`main`函数，进行调试，发现断点能命中，证明编译的文件以及`sourcemap`没有问题

![](https://unpkg.com/xiaochuan-static-dev@0.0.1/dist/0ddcdb3cfa852870.png)

### 后续

本章已经打包出`sourcemap`的产物并验证了可以调试源码，后续将使用此产物调试。可以结合`vscode`的`launch.json`来进行调试。

author: [xiaochuan](https://github.com/2239559319)

date: 2024.12.23
