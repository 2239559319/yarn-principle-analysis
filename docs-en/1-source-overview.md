# chapter 1 yarn overview

> The article will publish on [medium](https://medium.com/@w2239559319/list/yarn-principle-analysis-ebdbd4b1ab25)

> The source code of the book is located at [github](https://github.com/2239559319/yarn-principle-analysis)

> The `yarn` mentioned in the book refers to `yarn1 version`, If it is `yarn2`, `yarn3` or `yarn4`, the version number will be specially marked.

## Introduction to yarn

`yarn` is an old package management tool, and the latest `yarn4` has been in the iteration stage. After 2020, many new package management tools, such as `pnpm` and `yarn2` (also known as Berry), have developed, but yarn is still a very powerful package management tool now (2024). Many new package management tools, such as `pnpm`, have adopted the design and concepts of `yarn`. The more well-known ones are `resolution` and `patch`. These concepts are still used by various package management tools today, which shows that `yarn` is a very powerful tool.

Understanding the principles of `yarn` helps you understand the basic concepts of various package management tools and become proficient in using them. For understanding the principles of `yarn`, the book uses source code analysis to show how `yarn` works and runs. In addition, the book will mention some new package management tools such as `pnpm` and `yarn4`.

## Why choose Yarn

There are many package management tools, so why choose to talk about `yarn` in this book?

For new package management tools, such as `pnpm` and `yarn4`, the entire source code is too large and contains too much content. The source code is organized in the form of `monorepo`, and a lot of key information is not obvious.

Compared with `npm`, many of the codes related to `npm` are dependent on external packages, and the source code is not in the same repository, which makes debugging very troublesome.

In contrast, the contents of `yarn` are all in one repository [github/yarn/yarn](https://github.com/yarnpkg/yarn), and `yarn` has had basically no new commits in the past four years. It has stopped iterating for a long time and is iterated on `berry`, which is very conducive to debugging.

## The history of yarn

According to the [yarn official website blog](https://classic.yarnpkg.com/lang/en/), `yarn` was first released in 2016, and its initial focus was speed. It maintained frequent iterations until 2020. Starting in 2020, the `yarn` team mainly invested in the development of `yarn2` at the time, which was the `berry` version, which is the predecessor of the current latest version of `yarn4`. Judging from the [github/yarn repository](https://github.com/yarnpkg/yarn), `yarn` has not iterated since 2020. It only submitted a few minor configuration issues, and the remaining `pr`s were not merged. Now all features and bugs are being modified in the `berry` [repository](https://github.com/yarnpkg/berry).

As of 2024.12.26, the latest version of `yarn` is `1.22.22`.

## Source code debugging

Next is the guide to debugging source code.

### Environment Preparation

The environments that need to be prepared are `node`, `yarn`, and `git`.

Run the following command

```bash
git clone https://github.com/yarnpkg/yarn.git

cd yarn

git checkout -b mybuild-v1.22.22

git reset --hard v1.22.22

yarn
```

Since the package management tool of the `yarn` source code is still `yarn`, `yarn` is required in the prepared environment. Many package management tools also have their own package management tools for source code. For example, the source code of `pnpm` is managed by `pnpm`, and the source code of `npm` is also managed by `npm` itself.

### Code Analysis

Through `package.json`, we can see that the source code of `yarn` is written by `flow`. `Babel` and `babel plugin of flow` are used to complete the compilation. In order to unify the process, `yarn` uses `gulp`. The commonly used compilation is `build script`, which compiles to the `lib` directory, and this type of compilation is not bundled. Another type is `build-bundle`, which uses `webpack` to put all `yarn` codes and dependent codes into one file. `corepack` uses `yarn` which uses this form of packaging.

`yarn` has some dependencies such as `debug` and `commander` declared in `package.json`. These dependencies will be installed together with the global `node_modules` folder when `yarn` is fully installed. In `build-bundle` mode, they are packaged into a separate file together with the `yarn` source code.

Dependency graph of yarn

![Dependency graph of yarn](https://unpkg.com/xiaochuan-static-dev@0.0.1/dist/bd002718baebd9a2.png)

The `babel` packaged by `yarn` is now a very old version. `yarn` uses `babel6`, and now the mainstream is `babel7`. The plugins of the two versions are incompatible and the package names are different.

For debugging, we need to compile a product file with the correct `sourcemap`. Here, we use the new `babel` version and `@babel/preset-flow` to compile. Since most of the syntax in the `yarn` source code is `es6`, there is no need for `@babel/preset-env`, and the `es6` syntax code is generated directly. At the same time, in order to compile the `module` to `commonjs`, we need to use `@babel/plugin-transform-modules-commonjs`.

### Compiling debug version

Install `babel` related packages through the above analysis.

```bash
yarn add -D @babel/core @babel/cli @babel/preset-flow @babel/plugin-transform-modules-commonjs
```

Create a new `babel.config.js` configuration file.

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

Write compiled script.

```json
{
  "mybuild": "babel --no-babelrc src -s -d mybuild"
}
```

Run `yarn mybuild` or directly run `yarn babel --no-babelrc src -s -d mybuil`. We can see that the product file with `sourcemap` is generated in the `mybuild` directory.

Running `node mybuild/cli/index.js --version` outputs `1.22.22`.

![yarn --version output](https://unpkg.com/xiaochuan-static-dev@0.0.1/dist/806cfe4b473301ba.png)

Enter the `main` function of `src/cli/index.js` for debugging and find that the breakpoint can be hit, proving that there is no problem with the compiled file and `sourcemap`.

![debug](https://unpkg.com/xiaochuan-static-dev@0.0.1/dist/0ddcdb3cfa852870.png)

### Follow-up

This chapter has compiled the `sourcemap` product and verified that the source code can be debugged. This product will be used for debugging later. We can use `launch.json` of `vscode` for debugging.

author: [xiaochuan](https://github.com/2239559319)

date: 2024.12.23
