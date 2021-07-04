---
title: 【基于Git hooks搭建前端工作流系列】二、使用js实现代码语法检测
date: 2021-06-18 15:48:42
categories:
  - 前端
  - 工程化
tags:
  - Git
  - 工程化
description: 【基于Git hooks搭建前端工作流系列】的第二篇文章，主要介绍如何使用js实现git脚本文件，完成代码语法检测功能
---

## 一、前言

这是`基于Git hooks搭建前端工作流系列`的第二篇文章，主要介绍如何使用`js`实现`git`脚本文件，完成代码语法检测功能。

整个系列文章的目录如下：
一、方案选型：介绍现在主流的自动化方案。
二、使用 `js` 实现代码语法检测：介绍如何利用 `nodejs api` 做`文件 diff` 和`代码检测`。
三、引入插件化，集成和定制更多自动化功能：介绍如何将原来的脚本文件进行插件化，方便拓展功能和切换模板。

## 二、脚本解释器简介

以`git`提供的默认脚本文件`pre-commit.sample`的首行代码为例：

```
#!/bin/sh
```

这行代码用到了`Shebang`的概念，`wiki`给出了如下的介绍：

> 在计算领域中，Shebang（也称为 Hashbang）是一个由井号和叹号构成的字符序列#!，其出现在文本文件的第一行的前两个字符。 在文件中存在 Shebang 的情况下，类 Unix 操作系统的程序加载器会分析 Shebang 后的内容，将这些内容作为解释器指令，并调用该指令，并将载有 Shebang 的文件路径作为该解释器的参数。

从上面的描述可以看出，程序加载器会根据`Shebang`使用不同的解释器。所以我们开发`node`脚本时，可以在文件的头部增加对应的指令调用`node`解释器，如下所示：

```
#!/usr/bin/env node
```

以打印`Hello World`为例，我们先新建一个`node`脚本，命名为 `test`。将其设置为可执行权限，首行暂时不加`#!/usr/bin/env node`。此时在 shell 文件中执行 test 文件，会发现执行错误。
![错误脚本执行展示](2.png)
这就是因为不加`#!/usr/bin/env node`，识别不了`js`的语法。然后我们加上`#!/usr/bin/env node`再试试：
![正确脚本执行展示](3.png)
之所以使用`/usr/bin/env`，是因为不同用户的`node`解释器会安在不同的目录中，而`/usr/bin/env`会帮助我们去用户的`PATH`中依次查找`node`解释器的路径。

## 三、编写 node 版本的语法检测钩子

### 1、文件 diff，列出已修改的文件

在`shell`中可以通过以下命令，查看暂存区修改的文件：

```shell
// diff-filter：添加 (A), 赋值 (C), 删除 (D), 修改 (M), 重命名 (R)
git diff --cached --name-only --diff-filter=ACMR
```

- `--cached`：显示暂存区(已 add 但未 commit 的文件)和最后一次 commit ( HEAD )之间的所有不相同文件的增删改。
- `--name-only`：只显示名称。
- `--diff-filter=ACMR`：过滤掉已删除的文件，这些文件不需要做语法检测。

结合`node`的`api`，可以通过以下代码段，手动执行`shell`命令获取需要进行语法检测的文件名列表。

```js
const { exec } = require('child_process');

function getDiffFileList() {
  return new Promise((resolve, reject) => {
    exec('git diff --cached --name-only --diff-filter=ACMR', (error, stdout) => {
      if (error) {
        reject(`exec error: ${error}`);
      }

      resolve(stdout.split('\n').filter((diffFile) => /(\.js|\.jsx|\.ts|\.tsx)(\n|$)/gi.test(diffFile)));
    });
  });
}

getDiffFileList().then(console.log);
```

`demo`仓库的文件修改状况如下所示：
![result](7.png)
执行`shell`命令后的打印结果如下：
![diff](6.png)

### 2、eslint 检测文件

![Eslint](https://eslint.org/docs/developer-guide/nodejs-api#eslint-class)提供了`lintFiles`方法检测文件，我们将从上面程序中获取到的文件数组传给该方法，即可调用`eslint`进行语法检测。代码如下：

```js
const { ESLint } = require('eslint');

/**
 * @description 调用eslint api
 */
function lintFiles(fileList) {
  const eslint = new ESLint();
  return eslint.lintFiles(fileList);
}

/**
 * @description 整合并打印结果
 */
function output(resultList) {
  resultList.forEach((result) => {
    if (result.messages && result.messages.length > 0) {
      console.error(`ESLint has found problems in file: ${result.filePath}`);
      result.messages.forEach((msg) => {
        if (msg.severity === 2) {
          console.error(`Error: ${msg.message} in Line ${msg.line} Column ${msg.column}`);
        } else {
          console.warn(`Warning: ${msg.message} in Line ${msg.line} Column ${msg.column}`);
        }
      });
    }
  });
}

getDiffFileList().then(lintFiles).then(output);
```

最终打印结果如下：
![result](8.png)

### 3、增加 Shebang

首先，我们新建一份与`git hooks`的`pre-commit`钩子同名的`pre-commit`文件，注意该文件没有后缀。然后在`shell`中调用`chmod 777 pre-commit`将该文件设置为可执行文件，并在首行增加`#!/usr/bin/env node`。最后，我们将这段加了`#!/usr/bin/env node`的脚本拷贝到`git`仓库的`.git/hooks`目录中，就算大功告成了。
我们之后的每次提交，`git`都会触发该脚本对我们改动的文件进行语法检测。
最终的代码见 gist: [语法检测脚本](https://gist.github.com/Demian1996/d0dc27d08b38943cdb4eea212690c363#file-pre-commit)

## 四、总结

本文主要介绍了如何使用`js`实现`git`脚本文件，完成代码语法检测功能。同理，我们也可以使用`js`实现`commit 信息检测`，`npm packages 协议风险检测`等功能。
由于笔者团队没有使用`husky`，而是采用上述的**脚本文件 + vscode 插件注入的方式**（详细见该系列文章的第一篇）。随着业务的不断迭代，后续的开发者在维护脚本文件时，出现了维护困难、拓展困难的问题。在下一篇文章中，笔者将介绍如何使用插件化，来隔离**开发功能插件**和**组装 git 钩子业务**的关注点，并通过 vscode 插件实现动态切换当前项目需要使用的 git 钩子。

## 参考资料

- [用「增量」思想提升代码检查和打包构建的效率](https://juejin.cn/post/6865101730166767623)
