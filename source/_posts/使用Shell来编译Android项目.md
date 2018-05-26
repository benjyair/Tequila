---
title: 使用 Shell 来编译 Android 项目
layout: post
date: 2015/06/30 23:10:44
tags : Android
---

使用 Gradle 构建项目很久了，为了少敲命令行，我自己写了个简单的 Shell 脚本。

### 脚本
在项目目录下(不是 app moudle 目录下) 建一个 Shell 脚本 b.sh ，内容如下：

```python

#!/bin/sh
echo "清理缓存..."
./gradlew clean
echo "清理成功"
echo "开始编译..."
./gradlew build
echo "编译成功"
open ./app/build/outputs/apk/

```
注：如果你的 moudle 名字不是 app 那你就把脚本里面的 app 改名成你自己的 moudle 名字。

以后需要构建项目的话直接在 Terminal 里面敲 sh b.sh 就可以啦。

刚开始学 Shell ，脚本比较简单，不过也算是学以致用了。
