---
title: Android 简单实现圆形头像
layout: post
date: 2014/9/30 10:11:44
tags : Android
---

今天项目中即时通信工具部分要把好友头像由方形转换成圆形，大概也是跟上时代的潮流吧，毕竟连 QQ 都开始使用圆形头像了，我在这里记录一下修改过程。

在网上找了一番之后，大概解决方法有两种
> * 使用自定义组件继承 ImageView
> * 通过对 BitMap 重绘得到

第一种方式，有许多开源框架可以实现，如 [**CircleImageView**](https://github.com/nostra13/Android-Universal-Image-Loader)&nbsp;,由于我们项目中图片缓存及显示使用的是 [**Android-Universal-Image-Loader**](https://github.com/nostra13/Android-Universal-Image-Loader)，而他本身支持在获取到网络图片之后增加回调，那我毫无疑问的选择了第二种方式实现，

首先是圆形图片转换的方法

```java
/**
 * 将图片转为圆型  不标准的图形从中心截取
 *
 * @param bitmap
 * @return
 */
public static Bitmap getRoundedCornerBitmap(Bitmap bitmap) {
	int width = bitmap.getWidth();
	int height = bitmap.getHeight();
	Bitmap output = Bitmap.createBitmap(width, height, Config.ARGB_8888);
	Canvas canvas = new Canvas(output);
	// 切圆的直径
	int minLength = width > height ? height : width;
	final int color = 0xff424242;
	final Paint paint = new Paint();
	int x = (width - minLength) / 2;
	int y = (height - minLength) / 2;
	final Rect rect = new Rect(x, y, x + minLength, y + minLength);
	final RectF rectF = new RectF(rect);
	final float roundPx = minLength / 2;

	paint.setAntiAlias(true);
	canvas.drawARGB(0, 0, 0, 0);
	paint.setColor(color);
	canvas.drawRoundRect(rectF, roundPx, roundPx, paint);
	paint.setXfermode(new PorterDuffXfermode(Mode.SRC_IN));
	canvas.drawBitmap(bitmap, rect, rect, paint);
	return output;
}
```
再然后是显示图片工具类

```java
public static SimpleImageLoadingListener listener;

public static <T extends ImageView> void displayImage2Circle(T container, String url) {
	if (listener == null) {
		listener = new SimpleImageLoadingListener() {
			@Override
			public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
				super.onLoadingComplete(imageUri, view, loadedImage);
				((ImageView) view).setImageBitmap(getRoundedCornerBitmap(loadedImage));
			}
		};
	}
	ImageLoader.getInstance().displayImage(url, container, listener);
}
```
这样只要是需要把图片显示为圆形的地方直接调用 displayImage2Circle 这个方法即可


------

本以为已经大功告成，没想到使用之后却发现图片四周仿佛被切掉一部分一样，如图:

![tool-editor](http://oneylt1vv.bkt.clouddn.com/20140930151619.png)

> * 起先以为是图片有白边，后来发现不是，[图片地址](http://img0.bdstatic.com/img/image/shouye/mxzyq-11795342220.jpg)
> * 然后又以为是半径没算对,验证之后再次被排除
> * 最终把原因定位到了 Image-Loader 上,解决办法如下:

ImageLoaderConfiguration 的 defaultDisplayImageOptions 方法需要一个 DisplayImageOptions 参数而 DisplayImageOptions 的参数的 imageScaleType 属性默认的是 ImageScaleType.IN_SAMPLE_POWER_OF_2 就是这个属性导致的图片不圆,查了一下文档

```java
/**
 * 图片的缩放方式
 */
imageScaleType(ImageScaleType imageScaleType)
imageScaleType:
    EXACTLY :图像将完全按比例缩小的目标大小
    EXACTLY_STRETCHED:图片会完全缩放到目标大小
    IN_SAMPLE_INT:图像将被二次采样,并且倍数为整数倍
    IN_SAMPLE_POWER_OF_2:图片将降低2倍，直到下一减少步骤，使图像更小的目标大小
    NONE:图片不会调整
```

IN_SAMPLE_POWER_OF_2 模式下缩放是成倍缩放的所以导致实际显示的要比我设定的宽一些,改成 EXACTLY_STRETCHED 之后问题得到解决

最后展示一下最终效果

![tool-editor](http://oneylt1vv.bkt.clouddn.com/20140930154619.png)

以及 Image-Loader 的完整设置

```java
private void initImageLoader() {
	DisplayImageOptions options = new DisplayImageOptions.Builder()
    .showImageForEmptyUri(R.drawable.default_image)
    .showImageOnFail(R.drawable.default_image)
    .imageScaleType(ImageScaleType.EXACTLY_STRETCHED)
    .bitmapConfig(Config.RGB_565)
    .cacheInMemory(true)
    .cacheOnDisk(true)
    .resetViewBeforeLoading(true)
    .build();

	File cacheDir = StorageUtils.getOwnCacheDirectory(getApplicationContext(), Util.getCachePath());

	ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(this)
    .denyCacheImageMultipleSizesInMemory()
    .memoryCache(new LruMemoryCache(2 * 1024 * 1024))
    .threadPoolSize(4)
    .memoryCacheSize(2 * 1024 * 1024)
    .diskCache(new UnlimitedDiscCache(cacheDir)) // 缓存路径
    .diskCacheSize(50 * 1024 * 1024)
    .diskCacheFileCount(100)
    .defaultDisplayImageOptions(options)
    .build();

	 ImageLoader.getInstance().init(config);

}
```

公司项目我就不放源码了，核心代码都在上面了，有问题可以和我联系。
