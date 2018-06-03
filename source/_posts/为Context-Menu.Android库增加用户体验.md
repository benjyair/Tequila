---
title: 为 Context-Menu.Android 库增加用户体验
layout: post
date: 2015/01/25 22:01:53
tags : Android
---

Github 上面有很多的开源库，除了工具库之外，还有大量的UI库供我们使用，在使用这些UI库的时候开源者本人可能会给我们一些可以自定义的功能，可是在某种情况下，我们还需要对一些作者没有开放的功能进行定制，那我们就只能通过修改源码的方式来实现了，今天我介绍一下我对开源项目 Context-Menu.Android 的部分定制过程

### 介绍
[Context-Menu.Android](https://github.com/Yalantis/Context-Menu.Android) 是由Yalantis开源的一个上下文菜单的UI库，如下图，非常的酷炫。

![ContextMenu](https://d13yacurqjgara.cloudfront.net/users/125056/screenshots/1785274/99miles-profile-light_1-1-4.gif)

使用起来也非常的简单，具体使用大家可以在 Github 上看作者的使用介绍，我在这里就不在赘述。

### 问题
在使用这个库的时候，我遇到了如下问题：

当菜单打开的时候点击旁边灰色的部分该菜单 Menu 不能消失，只能点击任意一个 Item 或者点击返回键才能让其隐藏，那么我认为这样的体验是不好的。

### 解决
思路：给灰色背景布局加一个点击事件，该事件的处理就是 Menu 的点击处理，只不过不调用 Item 的回调方法，有了思路，那么找起来就简单的多了。

1.  首先我们需要找到完成点击动画的代码。打开 ContextMenuDialogFragment.java 我们可以轻易找到点击事件是在实现了 MenuAdapter.OnItemClickListener 接口的 onClick 方法中。那么我们进入 MenuAdapter.java，在这里我们终于找到了点击事件的代码。
![tool-editor](http://oneylt1vv.bkt.clouddn.com/20150125161035.png)
从代码中可以看出，只要我们给灰色的布局加上一个这个事件调用该方法就能达到我们的目的。

2.  其次我们需要找到灰色的布局组件。接下来进入该 Fragment 的布局文件 fragment_menu.xml 在这里我们发现了背景其实就是一个 RelativeLayout，那事情就好办多了，给这个 RelativeLayout 加上 id，在 ContextMenuDialogFragment 的 initViews 方 法中初始化一下该组件
![tool-editor](http://oneylt1vv.bkt.clouddn.com/20150125161418.png)

3.  最后 给 RelativeLayout 加上 OnClickListener 事件,然后调用之前关闭动画的那个方法。我的做法是直接给 MenuAdapter 新增加一个 dismissMenu 方法，去除无用的代码并指定默认点击的是第一个组件，因为出现的动画是由第一个 Item 开始的，所以这里把默认的执行动画由第一个来结束也最合理，代码如下
![tool-editor](http://oneylt1vv.bkt.clouddn.com/20150125161919.png)
![tool-editor](http://oneylt1vv.bkt.clouddn.com/20150125162038.png)

测试一下，大功告成

### 遗留问题
点击手机返回键的时候 Menu 是直接消失的，并没有执行结束动画，由于 Fragment 没有 OnKeyDown 方法，并且该组件继承自 DialogFragment，我在使尝试了几种办法之后最终无果，只好暂时放弃。等以后有时间再解决这个问题

项目的话由于涉及到开源协议，而我又不想弄分支所以就不发布修改版的库了，不过看上面的介绍大家也足可以做到我说的效果，还能帮助大家理解这个库，何乐而不为。如果实在有困难你可以联系我。
