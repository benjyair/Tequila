---
title: Android 简单实现圆形头像(续)
layout: post
date: 2014/10/11 21:16:25
tags : 技术积累
---

上一篇文章里面详细讲解了用自己的方式实现圆形头像，如今发现之前的代码效率并不高，因为每次都要 createBitmap 和转换图片，createBitmap 是一件很费内存的事，而频繁转换是一件很费 cpu 的事，这样费手机资源肯定必然是不行的，并且随着自己对 Imageloader 的深入使用，发现使用 Imageloader 还有另外一种更简单的方法来实现这样的效果，或许能解决这样的问题。

------

下面我详细介绍下，先看代码:

```java
 DisplayImageOptions options = new DisplayImageOptions.Builder()
    .showImageForEmptyUri(R.drawable.default_image)
    .showImageOnFail(R.drawable.default_image)
    .imageScaleType(ImageScaleType.EXACTLY_STRETCHED)
    .bitmapConfig(Config.RGB_565)
	.cacheInMemory(true)
	.cacheOnDisk(true)
	.resetViewBeforeLoading(true)
	.displayer(new RoundedBitmapDisplayer(112)).build();
```

这里面的 displayer 方法可以接收实现 BitmapDisplayer 接口的对象，关键点就在这里，Imageloader 有默认的可以实现圆角的 Displayer 那就是 FadeInBitmapDisplayer（int durationMillis），durationMillis 就是圆角的半径。用他就不用之前自己那么麻烦的方法了。

### 完整代码
Imageloader 的初始化方法没变，同上一篇 blog，这里只展示一下显示的工具类。

```java
//显示圆角的option
public static DisplayImageOptions options ;
	
public static <T extends ImageView> void displayImage2Circle(T container， String url) {
    if(options == null){
        options = new DisplayImageOptions.Builder()
            .showImageForEmptyUri(R.drawable.default_image)
            .showImageOnFail(R.drawable.default_image)
            .imageScaleType(ImageScaleType.EXACTLY_STRETCHED)
            .bitmapConfig(Config.RGB_565)
            .cacheInMemory(true)
            .cacheOnDisk(true)
            .resetViewBeforeLoading(true)
            // 我们的图片大小是70dp 所以我这里半径=70*1.6 ，其他情况可以自己算，或者动态设置
            .displayer(new RoundedBitmapDisplayer(112)) 
            .build();
    }
	ImageLoader.getInstance().displayImage(url, container, options);
}
```

### 附 :
其他 displayer

```java
displayer：  
        RoundedBitmapDisplayer（int roundPixels）设置圆角图片  
        FakeBitmapDisplayer（）这个类什么都没做  
        FadeInBitmapDisplayer（int durationMillis）设置图片渐显的时间  
        SimpleBitmapDisplayer()正常显示一张图片
```

### 疑问

Imageloader 缓存的图片是转换之前的 bitmap 呢还是之后的呢？
> * 如果是之前的，那执行效率并不会较之前我实现的方法提高多少，只是转换的工作交给了 Imageloader，每次还是要转换.
> * 如果缓存的是转换之后的圆角 bitmap 那效率就会大大提高，而我也希望是这样的，但这样拓展性可能会降低。

等有时间了我会深入研究一下 Imageloader 的源码，把结果写出来。
