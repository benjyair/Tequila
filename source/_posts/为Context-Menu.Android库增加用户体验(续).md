---
title: 为 Context-Menu.Android 库增加用户体验(续)
layout: post
date: 2015/02/28 16:40:22
tags : Android
---

在我的另一篇博客 **为Context-Menu.Android库增加用户体验** 里面留下了一个遗留问题就是给 DialogFragment 添加 OnKeyListener，当时朋友说这个很不好弄，加上当时项目比较紧自己也没有细细研究就把问题放了下来。今天在写 PubDialog 这个项目的时候又遇到了同样的问题，总不能再撂下吧，于是自己抱着试试的心态来解决这个问题，没想到这个问题原来如此简单。

### 解决

```java

@Override
public View onCreateView(LayoutInflater inflater,
             ViewGroup container, Bundle savedInstanceState) {

    /* do something */

    getDialog().setOnKeyListener(new DialogInterface.OnKeyListener() {
        @Override
        public boolean onKey(DialogInterface dialog, int keyCode,
                             KeyEvent event) {
            /* 处理 */
            return false;
        }
    });
    return rootView;
}

```

就是如此简单！！

### 拓展
该 Dialog 有大片的空白区域我顺便在下方做了一个小的 Tip 区，每次取出一条提示在里面展示。
如图：

![tool-editor](https://blog-1251733178.cos.ap-beijing.myqcloud.com/20150228181307.jpg)

同时把这个项目放在Github上了

[地址](https://github.com/BenjyAir/Sack)

需要的可以拿去
