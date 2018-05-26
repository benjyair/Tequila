---
title: 自己修改的 GustureLock
layout: post
date: 2015/03/04 21:40:39
tags : 技术积累
---

今天分享一个我修改过样式的 GustureLock 的源码，该库的出处我已经找不到了，当时是我朋友给我的一个 zip 包，我拿到源码之后，做了一些样式的调整，具体逻辑没有做处理，风格是模仿的 JDME 软件的风格

### 使用

在需要检查手势锁的地方加入如下代码，考虑到会需要随时修改切换动画，我并没有吧跳转逻辑写死，而是添加了一个回调接口

```java
LockUtil.checkLock(this,new OnCheckLockCallback() {
    @Override
    public void onHasLockCallback() {
        Intent intent = new Intent(this, LockActivity.class);
        startActivityForResult(intent, GO_LOCK);
        overridePendingTransition(R.anim.mi_right_in, R.anim.mi_left_out);
    }

    @Override
    public void onUnHasLockCallback() {
        //do something
    }
});
```

注意：一定要使用 startActivityForResult 方法来启动，该 Activity 会返回不同的 resuldCode 来通知你如何处理；

```java
@Override
protected void onActivityResult(int requestCode, int resuldCode, Intent date) {
    if (requestCode == GO_LOCK && resuldCode == 404) {
        //无手势密码进不去而退出
        finish();
    } else if (requestCode == GO_LOCK && resuldCode == 200) {
        //解锁成功
    } else if (requestCode == GO_LOCK && resuldCode == 401) {
        //忘记密码重新登录
    } 
    super.onActivityResult(requestCode, resuldCode, date);
}
```

设置手势密码的跳转代码如下

```java
Intent i = new Intent(this, LockSetupActivity.class);
startActivity(i);
```

### 效果

![tool-editor](http://oneylt1vv.bkt.clouddn.com/20150304132203.jpg)

[源码地址](https://github.com/BenjyAir/Sack) 
