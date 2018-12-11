---
title: 在 Mac OS 上为 Android 设备刷机
layout: post
date: 2017/03/18 20:27:12
tags : Android
---

最近一段时间在研究 Android 的 AOSP，打算拿淘汰下来的 Nexus 刷机练练手，顺道也把吃灰的树莓派利用起来。本篇主要总结了一下刷机的步骤，在树莓派上编译 AOSP 等之后再写。

### 工具准备
在 Mac OS 上刷机需要使用 [Google SDK Platform Tools](https://developer.android.com/studio/releases/platform-tools)，下载并解压，然后将 **platform-tools** 目录添加到环境变量。

### 解锁 Bootloader

1、打开开发者选项，勾选上 **USB 调试** 和 **OEM 解锁**。
2、通过命令`adb reboot bootloader`进入 bootloader 模式。
3、执行命令`fastboot flashing unlock`，然后手机上会显示提示警告界面，选 **Yes**，就解锁成功了。如果没反应，可以再执行一次。
4、执行`abd reboot`重启手机。

### 刷入第三方 Recovery（以 TWRP 为例）

1、进入 [TWRP](https://twrp.me/) 官网，右上角 **Devices** 找到自己的设备 **不同型号的 TWRP 不共用，切记**，**Download Links** 下面随便选择一个下载，下载文件名如：twrp-recovery.img。
2、通过命令`adb reboot bootloader`进入 bootloader 模式。
3、执行命令`fastboot flash recovery twrp-recovery.img`刷入 TWRP。
4、执行命令`fastboot boot twrp-recovery.img`进入 TWRP。
5、TWRP 的功能可以自行摸索，这里主要讲一下 ADB Sideload 这个功能。在 TWRP 首页进入 **Advanced**，左下角点 **ADB Sideload**，**勾选双清**（一般都要勾选），**右滑 Sideload**，接下来在电脑上执行`adb sideload filename.zip`，文件内容可以是你需要刷入的任何 zip（刷机包、升级包、降级删除包、基带包、Root 包、软件包等等等等）。

### 刷入 Root 权限（以 Magisk 为例）（可选）

[Magisk](https://github.com/topjohnwu/Magisk) is a suite of open source tools for customizing Android, supporting devices higher than Android 5.0 (API 21). It covers the fundamental parts for Android customization: root, boot scripts, SELinux patches, AVB2.0 / dm-verity / forceencrypt removals etc.

1、下载 [Magisk](https://github.com/topjohnwu/Magisk/releases) releases 版本，如：Magisk-v11.1.zip。
2、进入 TWRP，按前文介绍进入到 **ADB Sideload** 这一步。
3、执行命令`adb sideload Magisk-v11.1.zip`，等待刷入完成。
4、重启手机，系统内 root 已经完成。
Ps：作为 Android 开发者，为了方便 push/pull 文件，还需要使用`adb root`命令，但此时还是不行的，解决办法看下一步。
5、下载 adbd-Insecure，文件名如：adbd-Insecure-v2.00.apk。
6、执行命令`adb install adbd-Insecure-v2.00.apk`安装 adbd-Insecure。
7、打开 adbd-Insecure 应用，勾选 `Enable insecure adbd`和`Enable at boot`。
8、重启手机，系统已经完全 root。

<br/>
