---
title: 一招搞定 IOS 屏幕适配
layout: post
date: 2015/05/15 23:57:19
tags : Swift
---

首先从一首歌说起，你还在，你还在，头悬梁，锥刺股，做做做做适配么？你还以为使用宏定义，纯代码编写，就能高枕无忧么？你还把分辨率转像素当成当成卖身资本么？你还在格子间码子冲刺过劳死么？ no no no no no no no no no no no no ~ ~ 。人生苦短，IOS 应用适配所有 iPhone 的梦想，你可以复制。------[挖掘机技术哪家强](http://www.bilibili.com/video/av1572046/)

### 一个图

![PubDialog](http://oneylt1vv.bkt.clouddn.com/20150515173315.png)

### 一段话

第一步，第二步，就不多说了，重点是第三部。一般先点 Add Missing Constraints 增加约束，加完之后就可以适配所有的设备了。如果你动了布局点 Update Constraints 更新约束，如果布局和约束不符点 Update Frames 更新布局，如果添加了新的组件之后接着点 Add Missing Constraints 添加新的约束即可。

### 结束

好了，就是这么简单（纯代码编写 UI 的请略过）。
