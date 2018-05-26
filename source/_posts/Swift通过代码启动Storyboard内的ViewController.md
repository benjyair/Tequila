---
title: Swift 通过代码启动 Storyboard 内的 ViewController
layout: post
date: 2015/03/13 23:10:09
tags : Swift
---

IOS 设计布局的方式有3种，1、代码编写，2、xib 文件，3、storyboard，三种方式的好坏及使用我就不细说了，网上资料很多，我这里主要说下我在使用 storyboard 中遇到的问题。

### 问题
storyboard 是 Apple 最新的设计界面的方式。第一次使用我就被他的功能所吸引，简直比 Android 的方便了不止 10 倍，集设计和交互，编码为一身，基本上都可以作为原型开发工具了。作为初学者，我在选择界面开发的时候果断的选择了storyboard，可是在使用他的时候出现了问题，问题是这样的，启动应用首先是闪屏，然后根据是否登录分别进入登录页或者主页。这个时候我通过普通的方式启动 ViewController 发现一片漆黑。即使在 storyboard 里面绑定了对应的 ViewController 也是如此。

### 解决
现如今网上 Swift 的资料甚是稀缺，这个问题我解决了好久，后来想到通过 xib 方式启动V iewController 的时候可以加一个 xibname 的属性来加载xib文件，那么有没有这样的一种方式也能启动 ViewController 的时候同时加载 storyboard 呢，后来我发现 OC 有这样的一种方式，我对着 OC 的代码又查了 Swift 的 API 发现果然不出我所料。具体代码如下：

```swift
var story =  UIStoryboard(name: "Main", bundle: nil)
var controller = story.instantiateViewControllerWithIdentifier("id") 
				as UIViewController
self.presentViewController(controller, animated: true, nil)
```
其中 id 为 UIViewController 在 storyboard 属性面板中的 Storyboard ID， 这样就可以实现一个 storyboard 文件设计所有的 UIViewController 了 在跳转的时候需要逻辑的时候就使用代码的方式，不需要逻辑的时候就直接使用 Segue。这样的方式适合一个人开发，或者前期设计的时候使用。因为所有的布局在一个文件里面多人操作提交的时候容易出错。不过好像新版的已经解决的 git 提交识别问题了。不过这也不是什么大问题啦。

### Ps
其实我在想到这样的解决方案的时候没有写的这么简单的，在弄了好久没有解决的时候我临时的方案是使用xib，类似 Android 的布局与代码解耦的方式类似。布局在 xib 文件里面，逻辑在 UIViewController 里面，这样也解决了本文的问题，只是有好东西不能用心里不舒服啊，终于在昨晚 3 点的时候灵光一闪想到了解决方法。今天一来测试，果然不出所料。至此我可以正式踏入 IOS 的大门啦，甚是欢喜。
