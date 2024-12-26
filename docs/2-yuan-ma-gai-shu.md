---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 2 源码概述

## yarn的发展历史

从[yarn官网blog](https://classic.yarnpkg.com/lang/en/)来看`yarn`最开始于2016年发布，最开始的主打的就是一个快。一直保持很频繁的迭代知道2020年。2020年开始`yarn`团队主要投入到当时`yarn2`的开发，也就是`berry`版本，也就是现在的最新版`yarn4`的前身。从[github/yarn仓库](https://github.com/yarnpkg/yarn)来看，`yarn`从2020年开始就不再迭代，只是提交了几个细小的配置问题，剩余的`pr`全部没有合入。现在所有的feature以及bug都在`berry`的[仓库](https://github.com/yarnpkg/berry)进行修改

截至2024.12.26，`yarn`的最新版本是`1.22.22`

## 源码概述

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
  presets: ["@babel/flow"],
  plugins: [
    [
      "@babel/plugin-transform-modules-commonjs",
      {
        lazy: () => true
      }
    ]
  ]
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

## 后续

本章已经打包出`sourcemap`的产物并验证了可以调试源码，后续将使用此产物调试。可以结合`vscode`的`launch.json`来进行调试。
