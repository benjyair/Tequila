---
title: 「群辉工作站」基于 Drone 的 Android CI 环境 - 准备 Android 编译镜像
layout: post
date: 2021/03/18 12:55:12
tags : Docker
---

### 写在前面
在之前的 `搭建 Drone CI 环境` 文章中我们的 Drone CI 环境已经搭建了起来，可以开始跑 CI 任务了。
那么接下来就是准备一个我们自己的 Android 环境，用来放在 Drone 脚本中执行自动编译。


### 前期调研
在开始之前，我一直在纠结我的基础镜像应该继承自哪里，首先最容易想到的就是继承自 Ubuntu，然后在内部配置好 Java、Android SDK、
Gradle，这样就只需要把代码挂载在里面编译即可，只是这样略为繁琐，相当于自己搭建了一套 Linux 下的编译环境，每一步都需要自己配置，该方案暂定为「方案一」。

于是我开始找别的路子，Docker Hub 上有现成的 Gradle 的基础镜像，并且该镜像有 Gradle 和 JDK 共存的版本，那么是不是我只需要在项目内部配置好 Android SDK 就可以了呢？好像可行！该方案暂定为「方案二」。
经过我自己的肉测，两种方法都可行，各位可以各取所需。

### 「方案一」搭建 Linux 编译环境
该方案具体的执行就是纯体力活了，首先在本地 Docker 中启动一个 Ubuntu 基础镜像，然后一个一个的添加命令，最终把命令合集整理到 Dockerfile 中去，就出来了下面这个 Dockerfile 文件，虽然踩坑比较多，但是只要够细心就可以解决全部问题。
关于脚本中的 Gradle 是否需要下载，经过我的测试，即使不下载，我们仍然可以使用项目下的 gradlew 命令来自动下载 Gradle 依赖。只是这样比较耗时。我们在镜像中提前下载好，在每一次执行脚本的时候就不需要二次下载，变相的节省了项目编译的时间，所以还是推荐提前安装好对应的 Gradle 版本。
其中的环境变量 Android SDK 版本和 Build Tools 版本也是一样， 提前下载好我们项目对应的 SDK 版本，能大大节省执行脚本所需要消耗的时间。

```dockerfile
FROM ubuntu:20.04
LABEL author=benjyair site=https://www.benjyair.com/


# 准备环境与工具链
RUN apt-get update \
    && apt-get install sudo -y \
    && sudo apt-get install curl unzip -y \
    && sudo apt-get install build-essential file apt-utils -y

ENV JAVA_URL="https://code.aliyun.com/kar/oracle-jdk/raw/3c932f02aa11e79dc39e4a68f5b0483ec1d32abe/jdk-8u251-linux-x64.tar.gz" \
    JAVA_VERSION="jdk1.8.0_251" \
    GRADLE_URL="https://downloads.gradle.org/distributions/gradle-6.7-bin.zip" \
    GRADLE_VERSION="gradle-6.7" \
    SDK_URL="https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip" \
    ANDROID_SDK_ROOT="/usr/local/android-sdk" \
    ANDROID_VERSION=30 \
    ANDROID_BUILD_TOOLS_VERSION=30.0.2


# 安装 Java 8
RUN mkdir /usr/local/java \
    && cd /usr/local/java \
    && curl -o "$JAVA_VERSION.tar.gz" $JAVA_URL \
    && tar -zxvf "$JAVA_VERSION.tar.gz" \
    && rm "$JAVA_VERSION.tar.gz"

ENV JAVA_HOME="/usr/local/java/$JAVA_VERSION" \
    JRE_HOME="/usr/local/java/$JAVA_VERSION/jre"

ENV CLASSPATH=$JAVA_HOME/lib:$JAVA_HOME/jre/lib \
    PATH="${PATH}:${JAVA_HOME}/bin:${JRE_HOME}/bin"


# 安装 Android SDK
RUN mkdir -p ~/.android \
    && touch ~/.android/repositories.cfg \
    && mkdir "$ANDROID_SDK_ROOT" .android \
    && cd "$ANDROID_SDK_ROOT" \
    && curl -o sdk.zip $SDK_URL \
    && unzip sdk.zip \
    && rm sdk.zip \
    && mkdir "$ANDROID_SDK_ROOT/licenses" || true \
    && echo "24333f8a63b6825ea9c5514f83c2829b004d1fee" > "$ANDROID_SDK_ROOT/licenses/android-sdk-license" \
    && $ANDROID_SDK_ROOT/tools/bin/sdkmanager --update \
    && $ANDROID_SDK_ROOT/tools/bin/sdkmanager "build-tools;${ANDROID_BUILD_TOOLS_VERSION}" \
    "platforms;android-${ANDROID_VERSION}" \
    "platform-tools"

ENV PATH="${PATH}:${ANDROID_SDK_ROOT}/tools:${ANDROID_SDK_ROOT}/platform-tools"


# # 安装 Android NDK
# RUN curl http://dl.google.com/android/ndk/android-ndk-r10e-linux-x86_64.bin    && \
#     chmod a+x android-ndk-r10e-linux-x86_64.bin                                && \
#     ./android-ndk-r10e-linux-x86_64.bin -o/usr/local                           && \
#     rm android-ndk-r10e-linux-x86_64.bin
# # 配置 Android NDK 环境变量
# ENV NDK_HOME /usr/local/android-ndk-r10e
# ENV PATH $PATH:$NDK_HOME


# 安装 Gradle，加速编译 APK 过程，也可以使用项目内的 Gradle Wrapper 自动下载当前项目的需要的 Gradle 版本
RUN mkdir /usr/local/gradle \
    && cd /usr/local/gradle \
    && curl -o "$GRADLE_VERSION.zip" $GRADLE_URL \
    && unzip "$GRADLE_VERSION.zip" \
    && rm "$GRADLE_VERSION.zip"

ENV GRADLE_HOME="/usr/local/gradle/$GRADLE_VERSION"
ENV PATH="${PATH}:${GRADLE_HOME}/bin"

# 卸载工具包
RUN sudo apt-get autoremove curl unzip -y
```

镜像准备完毕后开始 Build 与 Push，由于我们的 Drone 安装在远端的群辉中，所以我们需要把我们的镜像提交到 Docker Hub 或者自己的私服中供 Drone 下载。

```shell
docker build -t benjyair/android-env .
docker push benjyair/android-env
```

在 Drone 的文档中讲到，Drone 会自动的 Clone 当前项目的代码到 `/drone/src` 下，那么这个 Job 就简单多了，以下是对应的 `.drone.yml` 文件。

```yaml
kind: pipeline
type: docker
name: android

steps:
  - name: build
    image: benjyair/android-env
    commands:
      - gradle assembleDebug
```

代码提交后，我们的 Drone 便会自动分配编译任务，虽然知道第一次编译会比较慢（下载镜像需要时间，群辉本身性能一般等），但是万万没想到足足花了 33 分 50 秒才完成编译。看到这个结果我不知道该高兴还是伤心，高兴的是终于成功了，伤心的是这速度搞下来有什么用？
好在，第二次测试只花了 5 分 08 秒，可以接受。证明「方案一」可行，那么接下来需要将编译完成的 APK 上传到文件服务器即可！这部分留在下一篇介绍。

### 「方案二」使用现有的 Gradle 镜像
本着不重复制造轮子的想法我也同样研究了方案二（其实是先研究了这个方案只是走了弯路 -_-!），以下是对应的 Dockerfile。

```dockerfile
FROM gradle:6.7-jdk8
LABEL author=benjyair site=https://www.benjyair.com/


USER root
# Install Build Essentials
RUN apt-get update \
    && apt-get install build-essential file apt-utils -y

ENV SDK_URL="https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip" \
    ANDROID_SDK_ROOT="/usr/local/android-sdk" \
    ANDROID_VERSION=30 \
    ANDROID_BUILD_TOOLS_VERSION=30.0.2

# Download Android SDK
RUN mkdir -p ~/.android \
    && touch ~/.android/repositories.cfg \
    && mkdir "$ANDROID_SDK_ROOT" .android \
    && cd "$ANDROID_SDK_ROOT" \
    && curl -o sdk.zip $SDK_URL \
    && unzip sdk.zip \
    && rm sdk.zip \
    && mkdir "$ANDROID_SDK_ROOT/licenses" || true \
    && echo "24333f8a63b6825ea9c5514f83c2829b004d1fee" > "$ANDROID_SDK_ROOT/licenses/android-sdk-license"

# Install Android Build Tool and Libraries
RUN $ANDROID_SDK_ROOT/tools/bin/sdkmanager --update \
    && $ANDROID_SDK_ROOT/tools/bin/sdkmanager "build-tools;${ANDROID_BUILD_TOOLS_VERSION}" \
    "platforms;android-${ANDROID_VERSION}" \
    "platform-tools"
```

接下来构建镜像、上传、在本地测试。

```shell
docker build -t benjyair/android-env-gradle .
docker push benjyair/android-env-gradle
# 切换到项目根目录执行以下命令
docker run --rm -v "$PWD":/home/gradle/ -w /home/gradle/ benjyair/android-env-gradle gradle -PdisablePreDex assembleDebug
```
很快，我们本地的项目就编译完成了，本机测试通过。接下来推送到 Drone 中，原本我是使用 dind（Docker in Docker） 在 Drone 中执行这个 Docker 命令，这里卡了我好久，后来我突然发现根本不需要绕弯路使用 dind，直接使用和「方案一」一样方法就可以。
以下是 `.drone.yml`文件。

```yaml
kind: pipeline
type: docker
name: android

steps:
  - name: build
    image: benjyair/android-env-gradle
    commands:
      - gradle assembleDebug
```
推送到 Git 仓库，速度还可以，第一次 7 分钟完成了编译，之前的 30 分钟估计是网络出现了问题。后来经过几次对比测试，两种方案结果相差不大，基本都是在 4 分钟到 5 分钟之内完成编译。「方案一」甚至要比「方案二」快 50 秒左右。

### 写在最后
本文使用设备：群辉 DS720+
Android SDK 版本：30
Android Build Tools 版本：30.0.2
Drone 和 Gitea 的搭建参见该系列之前的文章。
两个镜像都可以供大家测试
```shell
# 方案一继承自 Ubuntu
docker pull benjyair/android-env:latest
# 方案二继承自 Gradle
docker pull benjyair/android-env-gradle:latest
```
至此，整个 CI 步骤完成了 90%，最后还差 APK 上传功能。
