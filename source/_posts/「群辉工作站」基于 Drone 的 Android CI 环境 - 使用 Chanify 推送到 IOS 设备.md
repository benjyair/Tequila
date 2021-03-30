---
title: 「群辉工作站」基于 Drone 的 Android CI 环境 - 使用 Chanify 推送到 IOS 设备
layout: post
date: 2021/03/29 19:50:12
tags : Docker
---

### 写在前面
在 Android CI 项目的最后，还想再为该项目增加一个推送功能，编译完成后将结果通过 Chanify 推送到 IOS 设备。为什么选择 Chanify 呢？只是因为这天我刚好在 V2EX 上刷到了这篇[文章](https://www.v2ex.com/t/765999#reply21)，Chanify 所需要的条件我也都满足。而我的目的，Chanify 也能帮我完成。暂时撇开 Chanify 不讲，我们实际需要的是一个通知功能，能把消息送到指定的平台即可。目前 Drone 已经支持了许多的[插件](http://plugins.drone.io/)，其中包括[钉钉](http://plugins.drone.io/lddsb/drone-dingtalk-message/)、[微信](http://plugins.drone.io/lizheming/drone-wechat/)、[Slack](http://plugins.drone.io/drone-plugins/drone-slack/)、[ServerChan(Server酱)](http://plugins.drone.io/yakumioto/drone-serverchan/)。除去这些现成的，我们完全可以基于 Docker 做出任何自定义的镜像来实现消息的推送。

### 准备 Chanify 镜像
关于 Chanify 的使用可以参考[这里](https://github.com/chanify/chanify)，我这里选择的是「作为有状态服务器」模式，在群辉中搭建一个 Chanify 服务器，通过 `Drone => 自建节点服务器 => Apple server => Chanify iOS客户端` 来完成整条回路。以下是对应的 docker-compose 文件：
```yaml
version: '3'

services:
  chanify:
    image: wizjin/chanify:latest
    container_name: chanify
    restart: always
    volumes:
      - ./data:/root/.chanify
    ports:
      - "9070:80"
    command: serve --name=<nas> --endpoint=http://192.168.192.1:9070
```
还是通过 docker-compose 的方式启动容器，启动完成后访问 `http://192.168.192.1:9070/` 获取节点二维码，然后在 App Store 下载 Chanify 客户端，在节点 Tab 扫描二维码完成节点绑定，这里需要注意的是，这个时候客户端需要访问这个内网地址，所以绑定的时候 IOS 设备需要与群辉（Chanify主机）在同一局域网，之后就无所谓了。绑定完成后到频道 Tab 任选一个频道获取到我们节点的 Token，将该 Token 写入 Drone 项目的 Secret 中，至此，前期准备完毕。

### 准备 Drone 通知脚本
在之前的文章中，我们的 Drone 脚本已经将 APK 上传到了 MinIO，现在我们在其之后增加推送的步骤，直接看最终的 `.drone.yml` 配置：
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

  - name: notify-success
    image: curlimages/curl
    environment:
      CHANIFY_TOKEN:
        from_secret: chanify_token
    commands:
      - cd app/build/outputs/apk/eng/release
      - apk_file=`ls | head -1`
      - curl --form-string "text=Build and upload Success. MinIO[$apk_file]" "http://192.168.192.1:9070/v1/sender/$CHANIFY_TOKEN"

  - name: notify-failure
    image: curlimages/curl
    environment:
      CHANIFY_TOKEN:
        from_secret: chanify_token
    commands:
      - curl --form-string "text=Build and upload failure." "http://192.168.192.1:9070/v1/sender/$CHANIFY_TOKEN"
    when:
      status:
        - failure
```
以上：
1、`notify-success` 在成功时调用，通过 curl 镜像完成 web 调用，将结果通知到 Chanify。
2、`notify-failure` 通过 `status=failure` 来控制只在失败的时候调用，通知 Chanify 编译失败，若编译成功则会跳过该步骤。
3、`chanify_token` 是前面我们配置在 Drone 的 Secret 中的频道 Token。

提交修改后的 `.drone.yml` 配置文件到 Git 仓库，安心的等着手机收到执行结果的推送。

### 最后
再一次感慨 Docker 的好用，方便，简单！
<br/>
