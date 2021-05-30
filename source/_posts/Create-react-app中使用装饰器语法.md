---
title: Create-react-app中使用装饰器语法
date: 2019-03-01 02:29:45
description: 本文主要介绍了如何在cra项目中拓展使用装饰器语法，从而支持mobx的装饰器。
categories:
  - 前端
  - 其他
tags:
  - React
  - Babel
  - Mobx
  - 装饰器
cover: /2019/03/01/Create-react-app中使用装饰器语法/index.png
---

### 背景：

想要在 create-react-app 中使用 mobx，因为装饰器模式看起来简单，所以想让 create-react-app 支持装饰器语法。
### 项目搭建

```shell
create-react-app test-decorator
npm install -s mobx-react mobx
```

安装完毕后，若直接在 class 中使用 mobx 的@observer 语法

```jsx
import {@observer} from 'mobx-react';
@observer
class Demo
```

此时会报编译错误
<!--more-->

![1551331520473](1551331520473.png)

所以需要使用 babel 让编译时能支持装饰器语法。

> ps：create-react-app 可以使用其他方式支持装饰器语法，这里只提供一种简单粗暴的形式-弹出。

### 弹出 create-react-app 的项目

```shell
npm run eject
```

发现出了点小意外:

mobx 不应该先装，`eject得在工作区没有改动时启用，不然会弹出失败。`

于是先提交工作区的改动

```shell
git commit -a -m "init"
```

然后再次 eject

### 安装`@babel/plugin-proposal-decorators`

```shell
npm install --save-dev @babel/plugin-proposal-decorators
```

在`package.json`中加上

```js
{
  "plugins": ["@babel/plugin-proposal-decorators"]
}
```

此时运行报错

![1551335060987](1551335060987.png)

加上`decoratorsBeforeExport`：

```json
["@babel/plugin-proposal-decorators", { "decoratorsBeforeExport": true }],
```

报错：

![1551335316388](1551335316388.png)

改成

```json
{
  "plugins": [
    ["@babel/plugin-proposal-decorators", { "legacy": true }],
    ["@babel/plugin-proposal-class-properties", { "loose": true }]
  ]
}
```

成功!!!

猜测和我的下面的写法有关，装饰器挂在头部怪怪的

```
@a
class B
```

好像大佬们认为下面这样更好一点

```js
export @a class B
```
