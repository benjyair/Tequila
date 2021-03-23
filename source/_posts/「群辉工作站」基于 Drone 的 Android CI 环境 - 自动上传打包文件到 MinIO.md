---
title: 「群辉工作站」基于 Drone 的 Android CI 环境 - 自动上传打包文件到 MinIO
layout: post
date: 2021/03/23 18:25:12
tags : Docker
---

### 写在前面
在之前的文章中我们完成了 CI 环境的大部分工作，已经实现了提交代码自动编译，本文主要完成最后一步，上传编译后的 APK 到 MinIO。


### 准备 MinIO
[MinIO](https://docs.min.io/cn/) 是一个非常轻量的对象存储服务，可以轻松快速的部署在 Amazon S3 或者 Docker 集群中。

```yaml
version: '3'

services:
  minio:
    image: minio/minio:latest
    container_name: minio
    restart: always
    volumes:
      - ./data:/data
    ports:
      - "9090:9000"
    command: server /data
    environment:
      - MINIO_ROOT_USER=benjyair
      - MINIO_ROOT_PASSWORD=abc123
```
依旧使用 compose 启动
```shell
# 1、在 File Station 中上传 docker-compose.yml 文件到 docker 共享文件夹的 minio 目录，
#    如果没有就自己创建。我的 docker 共享文件夹在 存储池2 上，所以 compose 文件的路径就是：
#    /volume2/docker/minio/docker-compose.yml
# 2、ssh 登入群辉，ssh 用户名@群辉IP
ssh benjy@192.168.192.1 
# 3、获取 root 权限，进入 compose 路径
sudo -i  # 再输入一次当前用户密码就行
cd /volume2/docker/minio/
# 4、创建容器
docker-compose up -d
```
之后通过 `http://192.168.192.1:9090` 访问 MinIO，登录后首先为该项目创建一个叫 `draque` 的 Bucket 。Bucket 在 MinIO 中类似于项目的意思，也可以理解为项目根目录。之后我们自动打包的 APK 都放在这个 Bucket 下。

<br/>
MinIO Quickstart Guide:
```text
Object API (Amazon S3 compatible):
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
```

### 准备 Android 项目的配置文件
既然需要自动化，那项目的配置文件也需要正式起来，修改我们 Android 项目的 `build.gradle`，重命名输出的 APK 文件，这里我顺道加入了一些渠道配置。
```gradle
plugins {
    id 'com.android.application'
    id 'kotlin-android'
}

android {
    compileSdkVersion 30

    defaultConfig {
        applicationId "com.benjyair.darque"
        minSdkVersion 16
        targetSdkVersion 30
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    flavorDimensions "mode"
    productFlavors {
        official { dimension "mode" }
        eng { dimension "mode" }
    }

    android.applicationVariants.all { variant ->
        variant.outputs.all { output ->
            def outputFile = output.outputFile
            if (outputFile != null && outputFile.name.endsWith('.apk')) {
                def createTime = new Date().format("yyyyMMdd-HHmm", TimeZone.getTimeZone("GMT+08:00"))
                def fileName = "App-${defaultConfig.versionName}-${variant.productFlavors[0].name}-${createTime}.apk"
                outputFileName = fileName
            }
        }
    }
}
```
然后修改 `.drone.yml` 
```yaml
kind: pipeline
type: docker
name: android

steps:
  - name: build
    image: benjyair/android-env
    commands:
      - gradle assembleEngRelease

  - name: upload
    image: minio/mc
    environment:
      ACCESS_KEY:
        from_secret: minio_access_key
      SECRET_KEY:
        from_secret: minio_secret_key
    commands:
      - mc alias set minio http://192.168.192.1:9090 $ACCESS_KEY $SECRET_KEY
      - cd app/build/outputs/apk/eng/release
      - apk_file=`ls | head -1`
      - mc cp $apk_file minio/draque
```
以上文件中：
1、我修改了编译命令为 `assembleEngRelease`，这样只会只打包 Eng Release 版本。
2、上传的镜像使用的是 `minio/mc`，这个是 MinIO 的官方客户端镜像，其中提供了访问 MinIO 的 API。
3、MinIO 的用户名和密码使用了 Drone 的 Secret 机制，增加安全性，设置路径为 Drone 项目的「SETTINGS」--> 「Secrets」。
<br/>
提交到 Gitea 后，等 Drone Job 执行完成，在 MinIO 的控制台我们就能看到此次编译的 APK 已经完成了上传。

### 写在最后
至此，我就在群辉上完整的搭建了一套 Android CI 环境。磕磕绊绊，踩坑无数。系统的学习了 Docker 的使用、Docker 镜像的制作、Docker Compose 的使用，以及涉及到的和常用的一些 Docker 镜像，也了解了 Docker Machine、Docker Swarm 等集群的概念。还让在角落里的群辉发光发热，以上，真是值得开心和记录的事情。回想起 2013 年为了学习 OpenStack 而借了两台电脑搞集群的情景，不得不佩服 Docker 的强大，以及科技发展带来的便利性。
<br/>
除去项目中列出的配置，整个项目的源码我放在[这里](https://github.com/benjyair/Draque) ，需要的可以自取。
