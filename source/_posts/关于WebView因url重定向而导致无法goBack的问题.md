---
title: 关于 WebView 因 url 重定向而导致无法 goBack 的问题
layout: post
date: 2014/11/11 23:13:50
tags : Android
---

最近项目中有一些界面需要嵌入 wap 页，在按返回键的时候让 WebView goBack, 大部分界面都是可以正常回退的,可是某些会重定向的地址却无法正常 goBack，原因是，退回重定向之前的 url 又被重定向了回来,网上的解决办法是自己控制一个 url 集合,我试了一下非常麻烦,因为我们还需要 goForward 功能,仅仅使用一个 LinkedList 还满足不了需求.最终还是放弃了这样的做法,后来终于在 stackOverflow 上找到了解决的办法,解决的方法真是格外的简单.直接看代码

为了让 WebView 控制界面里面的 url 跳转,我们一般都会设置 WebViewClient ，并重写 shouldOverrideUrlLoading 方法，让 WebView 加载点击 url，一般的例子代码如下:

```java
webView.setWebViewClient(new WebViewClient() {

	@Override
	public boolean shouldOverrideUrlLoading(WebView view, String url) {
		 view.loadUrl(url);
		return true;
	}
});
```

解决办法:

```java
webView.setWebViewClient(new WebViewClient() {

	@Override
	public boolean shouldOverrideUrlLoading(WebView view, String url) {

		return false;
	}
});
```

万万没想到解决办法就在 api 里面,shouldOverrideUrlLoading这个方法的 api 如下:

Give the host application a chance to take over the control when a new url is about to be loaded in the current WebView. If WebViewClient is not provided, by default WebView will ask Activity Manager to choose the proper handler for the url. If WebViewClient is provided, return true means the host application handles the url, while return false means the current WebView handles the url.

就是说如果 application 处理这个 url 则返回 true,如果 WebView 处理这个 url 则返回 false.我们让 WebView 处理了这个 url 就应该返回 false,否则相当于处理的两(多)次,而这也是这个问题出现的原因所在.

后来我又看了一下 WebViewClient 的其他方法发现原来好多问题都可以在这里解决,只是之前用的不多不知道有这些方法,例如:

> * onPageStarted 在页面加载开始时调用.
> * onPageFinished 在页面加载结束时调用.
> * onLoadResource 在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次.

如果你还需要更丰富的处理效果那么推荐你用 WebChromeClient.


ps: 最近国内 github 好不稳定啊，大家还是翻墙吧.
