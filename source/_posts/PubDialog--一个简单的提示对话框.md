---
title: PubDialog--一个简单的提示对话框
layout: post
date: 2015/02/27 17:56:12
tags : Android
---

项目里需要做一个有多个选项的单选对话框，类似 QQ 的 Dialog 风格，实现起来倒是不难，只是这样的功能我在之前的项目里就写过一次，这次又要为一些小的改动再写一遍。干脆，我把这样的 Dialog 封装一下开源出来，也方便自己以后再用到的时候就不再需要做重复的工作。加上自己一直想写个开源项目，所以这次借着这股东风，PubDialog 诞生了。PubDialog 是一个类似 IOS 风格的对话框，我封装了一些简单的功能，方便在大部分场合快速实现选择对话框的功能。

### 使用

```java

//初始化一个字符数组
List<String> list = new ArrayList<>(3);
list.add("Send message");
list.add("Like profile");
list.add("Add to favorites");

//初始化PubDialogFragment
PubDialogFragment pubDialog = PubDialogFragment.newInstance(list, false);

//设置回调(也可以不设置)
pubDialog.setItemClickListener(new PubDialogFragment.ItemClickListener() {

    @Override
    public void onItemClick(View clickedView, DialogObject dialogObject,
    	int groupIndex, int itemIndex) {
    	//在回调中处理事件
        Intent intent;
        if (itemIndex == 1) {
            Uri uri = Uri.parse("https://github.com/KokerWang/PubDialog");
            intent = new Intent(Intent.ACTION_VIEW, uri);
        } else {
            Uri uri = Uri.parse("http://www.kokerwang.com");
            intent = new Intent(Intent.ACTION_VIEW, uri);
        }
        startActivity(intent);
    }
});

//在使用的地方
pubDialog.show(getSupportFragmentManager(), "setting");

```
### 功能定制

* 字体颜色
* 背景
* icon
* 多分组

更多定制功能请留意API

### 效果

![PubDialog](https://blog-1251733178.cos.ap-beijing.myqcloud.com/pubdialog_show_demo.gif)

* [项目地址](https://github.com/BenjyAir/PubDialog) (包含示例项目和lib库)

* [下载Demo](https://github.com/BenjyAir/PubDialog/blob/master/apk/PubDialogExample-debug.apk?raw=true)
