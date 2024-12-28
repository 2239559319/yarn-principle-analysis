# 调试指南

在上一章中已经打包出带有sourcemap的`yarn`产物，本章将初步进行调试环境的配置

## 使用vscode进行调试

新建一个文件夹作为调试文件夹，也就是真正运行`yarn`命令的地方。在`yarn`的源码文件夹新建`vscode`的`launch.json`。选择`node -> launch`，将其中的`cwd`选项设置为新建的调试文件夹的路径。

完整的配置文件如下

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "test yarn version",  // 这里只写了最简单的yarn --version命令的调试
      "skipFiles": ["<node_internals>/**"],
      "console": "internalConsole",
      "outFiles": [
        "${workspaceFolder}/**/*.(m|c|)js",
        "!**/node_modules/**"
      ],
      "program": "${workspaceFolder}/mybuild/cli/index.js",
      "cwd": "${workspaceFolder}/../yarn-source-dev",  // 这个设置为新建的调试文件夹的路径，相对路径绝对路径都可以
      "args": [
        "--version"
      ]
    }
  ]
}
```

断点到`src/cli/index.js`里面进行调试，发现能命中断点

![vscode调试](https://unpkg.com/xiaochuan-static-dev@0.0.3/dist/24ad19a882f6d7c4.png)

在接下来的调试里面通过增加`launch.json`里面`configurations`的来调试每个命令，只需把例子中的`args`修改成想调试的命令即可。

## node火焰图

### 生成node火焰图

生成`nodejs`的运行火焰图可以很明显的看到运行的堆栈以及调用的顺序。

在调试文件夹里面增加默认的包然后删除`node_modules`

```bash
yarn init -y

yarn add react@18   # 这里一定使用18，因为18有依赖更好分析install的过程

rm -rf node_modules
```

找到打包出的入口js文件位置，相对与yarn源码文件夹的路径为`./mybuild/cli/index.js`，复制绝对路径。运行下面的代码，会在调试文件夹里面生成一个cpuprofile文件，这就是火焰图的文件，这个文件可以用浏览器的调试工具打开。

```bash
node --cpu-prof --cpu-prof-interval=1 /yarn-dev/yarn/lib/cli/index.js install
```

运行后生成的`cpuprofile`文件

![运行后生成cpuprofile文件](https://unpkg.com/xiaochuan-static-dev@0.0.4/dist/a95adf7aeacbcb13.png)

### 查看火焰图

进入浏览器`chrome://inspect/`的`Open dedicated DevTools for Node`

![chrome://inspect](https://unpkg.com/xiaochuan-static-dev@0.0.4/dist/f4a11aa18da67739.png)

选中`performance`以及上传文件按钮

![上传文件](https://unpkg.com/xiaochuan-static-dev@0.0.4/dist/11890d221043fc72.png)

在文件选择里面选择刚才生成的cpuprofile文件，可以看到完整的调用栈信息的展示。

> 作者这里使用的是wsl，所以底下的文件没有映射到源代码里面，源代码的文件夹应该是`src`，这里因为使用了wsl导致`sourcemap`映射出了问题还是显示的打包后的文件夹`mybuild`。实际上使用`windows`或者`linux` `mac`都可以成功映射到源代码里面。这里关系并不大因为源码和打包后的代码里面的函数名基本没有改变所以也能看。

![选择文件后](https://unpkg.com/xiaochuan-static-dev@0.0.4/dist/cdfa0f0e06be932c.png)

## 总结

本章介绍了使用`vscode`对`yarn`的源码进行调试，以及使用`chrome`的调试工具对`yarn`命令生成的火焰图进行分析。在接下来的调试里面会使用这两个方法进行分析调试。`vscode`的调试主要使用断点进行调试，方便观察每一步的变量以及逻辑，而火焰图则方便观察命令整体的运行步骤以及运行规律。
