---
title: '简化B站UI,节约精力'
date: 2023-12-30 15:13:46
categories:
tags:
description:
---

## 背景

我是 B 站的重度用户，经常要在 B 站上看一些知识类的视频，一般是使用网页版。但是，使用网页版有个问题是经常会被 B 站的首页推荐、动态栏的红色标记和搜索框的热搜吸引走注意力，不自觉地浪费时间。因此，我希望简化 B 站的页面，让网页版的 B 站更契合我使用习惯的同时，减少大数据推荐对我的干扰。于是，我借助 chrome 的 stylish 插件配置了一个简化的 UI 界面。

## 效果对比

左边是简化后的，右边是原始页面。

### 首页

![首页](1.png)

可以看到各个分区、推荐都没有了，保证进入 B 站只能搜索自己需要的视频，去除一切干扰项。

### 搜索框

![搜索框](2.png)

移除了热搜，只保留了搜索历史，防止搜索的时候被热门词条吸引。

### 搜索结果页

![搜索结果页](3.png)

搜索结果页则是隐藏了头部导航条。因为登录账号后导航条的动态栏会有显示未读动态的小红点，让人忍不住点进动态页，看看关注的人新发的视频。
点进某个视频后，对应视频页的导航栏也被移除了，原因同上。

## 具体实现

### 一、安装 stylish 插件

![插件](4.png)

该插件可以让用户自定义 css 样式，并注入到特定的页面中。

### 二、新建自定义样式

进入 B 站主页后，点击 stylish 插件，此时会出现悬浮窗，切换到 editor 页面，如下图所示：

![demo](5.png)

1 是一组自定义 css 样式的名称，2 是代码块，每个代码块包含一组 3 和 4。其中 4 是自定义的 css 样式，3 则是样式生效的网址。
读者只需要按照以下配置拷贝到 stylish 中即可，标题是每个代码块的 domains。

#### www.bilibili.com/

```css
/* 屏蔽广告 */
.ad-report,
.video-page-special-card-small {
  display: none !important;
}
```

#### www.bilibili.com

```css
/* 工具栏可视 */
#i_cecream .bili-header,
#i_cecream .bili-header__bar {
  display: block !important;
}

/* 调整搜索栏样式 */
.bili-header__bar {
  background: var(--graph_bg_regular);
}

#nav-searchform {
  margin-top: 50vh;
  transform: translateY(-50%);
  background-color: white !important;
}

/* 屏蔽干扰项 */
.left-entry,
.trending,
.right-entry,
.bili-header__banner,
.bili-header__channel,
.header-channel,
.bili-feed4-layout,
.palette-button-wrap {
  display: none !important;
}
```

#### search.bilibili.com

```css
/* 屏蔽搜索热搜 */
.trending {
  display: none !important;
}
```

#### bilibili.com

```css
/* 屏蔽导航栏 */
#bili-header-container,
.bili-header,
.bili-header__bar,
#app #biliMainHeader {
  display: none !important;
}

/* 屏蔽底部 */
#biliMainFooter {
  display: none !important;
}
```

#### space.bilibili.com

```css
/* 屏蔽个人信息banner */
.h {
  display: none !important;
}

/* 屏蔽个人空间导航*/
/* #navigator {
	display: none !important;
} */

/* 收藏页的左侧导航*/
/* .fav-sidenav {
	display: none !important;
} */
```

### 三、应用样式

拷贝完成后点击保存，此时切换到`My styles`，将按钮切换到激活态即可应用。
![应用](6.png)

## 总结

本文介绍了利用 stylish 插件简化 B 站 UI，以防止自控力被大数据干扰项不断浪费的问题。B 站上有很多优秀的内容，但是也在不可避免的快餐化。读者可以基于这个插件，对其他常用的知识网站进行一定的简化，以保证日常工作流和学习流不会被无端分心。
