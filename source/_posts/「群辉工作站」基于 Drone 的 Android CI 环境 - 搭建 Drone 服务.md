---
title: 「群辉工作站」基于 Drone 的 Android CI 环境 - 搭建 Drone 服务
layout: post
date: 2021/02/26 19:50:12
tags : Docker
---

### 了解 Drone
在 Docker 的基础上，可选择 CI 工具比较多，选择 Drone 最主要还是源于其较低的资源开销，以及我也确实不喜欢 Jenkins。
最终目的是在群辉上搭建出一套自动构建 Android App 的 CI 平台。


### 准备好 Gitea
Drone 的使用需要一个 Git 仓库，Drone 支持 Github、Gitlab、Gogs、Bitbucket、Gitea 等，在线的的仓库需要 Drone 部署在公网上。
我暂时没有，所以在 Gogs 和 Gitea 中选择了 Gitea，这也是 Drone 官方推荐的，刚好也满足我在群辉上使用的需求。
<br/>
在 Gitea -> 「设置」 -> 「应用」 -> 「管理 OAuth2 应用程序」 中创建 drone OAuth 应用程序。
重定向 URL 设置为 `http://192.168.192.1:9080/login` 用来打通 Drone 与 Gitea。
另外获取到两个参数 CLIENT_ID 和 CLIENT_SECRET，为后面配置环境做准备。


### 创建容器
本文依旧使用 docker-compose 来创建容器。运行 Drone 共需要两个容器，`drone/drone` 和 `drone/drone-runner-docker`。

drone runner 负责向 drone server 轮询来执行具体的任务，随着 Drone 的版本发展，原本的 drone agent 已经改为 drone runner。
下面是具体的 compose 文件， 也可参考 [官网](https://docs.drone.io/server/provider/gitea/) 设置最小的环境参数。

docker-compose.yml
```yaml
version: '3'

services:
  drone-server:
    image: drone/drone:latest
    container_name: drone-server
    restart: always
    volumes:
      - ./data:/var/lib/drone
    ports:
      - "9080:80"
      - "9443:443"
    environment:
      - DRONE_OPEN=true
      - DRONE_DEBUG=true
      - DRONE_SERVER_HOST=192.168.192.1:9080
      - DRONE_GIT_ALWAYS_AUTH=false
      - DRONE_GITEA=true
      - DRONE_GITEA_CLIENT_ID=f9de217a-2835-4292-96c3-14e550dfd2c5
      - DRONE_GITEA_CLIENT_SECRET=MIjqGKSRJxtS4PdXpZ7iulvEBZ83McxaH0m7rp5AgWg=
      - DRONE_GITEA_SERVER=http://192.168.192.1:3000
      - DRONE_DATABASE_DATASOURCE=/var/lib/drone/drone.sqlite
      - DRONE_DATABASE_DRIVER=sqlite3
      - DRONE_SERVER_PROTO=http
      - DRONE_RPC_SECRET=b7dd563fac66927c8ca6bb6a777bb723
      - TZ=Asia/Shanghai
      - DOCKER_API_VERSION=1.39
  drone-runner:
    image: drone/drone-runner-docker:latest
    container_name: drone-runner
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - "9000:3000"
    depends_on:
      - drone-server
    environment:
      - DRONE_DEBUG=true
      - DRONE_RPC_HOST=192.168.192.1:9080
      - DRONE_RPC_SECRET=b7dd563fac66927c8ca6bb6a777bb723
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=drone-runner
      - DRONE_SERVER_PROTO=http
      - DOCKER_API_VERSION=1.39
```
`DOCKER_API_VERSION=1.39` 解决 runner 报错问题。
```text
deployment: Error response from daemon: client version 1.40 is too new. Maximum supported API version is 1.39
```

`DRONE_RPC_SECRET` 可以通过以下命令生成，注意 server 与 runner 的 DRONE_RPC_SECRET 需要一致。
```shell
$ openssl rand -hex 16
b7dd563fac66927c8ca6bb6a777bb723
```

进入 Shell 开始创建
```shell
# 1、在 File Station 中上传 docker-compose.yml 文件到 docker 共享文件夹的 drone 目录，
#    如果没有就自己创建。我的 docker 共享文件夹在 存储池2 上，所以 compose 文件的路径就是：
#    /volume2/docker/drone/docker-compose.yml
# 2、ssh 登入群辉，ssh 用户名@群辉IP
ssh benjy@192.168.192.1 
# 3、获取 root 权限，进入 compose 路径
sudo -i  # 再输入一次当前用户密码就行
cd /volume2/docker/drone/
# 4、创建容器
docker-compose up -d
```
到这里容器就创建好了，去 Docker 套件中检查容器是否创建完成并启动成功，并检查日志中是否存在异常。

### 初始化 Drone
打开浏览器访问 Drone 服务：`http://192.168.192.1:9080/`

这里默认会跳转到 Gitea 进行应用授权，授权后返回 Drone 便能看到 Gitea 中创建的仓库。
我们提前在 Gitea 中创建一个 demo 项目，同时在 Drone 中点击 ACTIVATE，runner 会自动为该项目创建对应的 webhook。
接下来提交一个 .drone.yml 配置文件到 Git 仓库，配置中我们只是简单的打印出当前环境信息，
然后用 5 次 ping 模拟延时。提交完成后 Drone 就会自动执行该 pipeline。

.drone.yml
```yaml
kind: pipeline
type: docker
name: default
steps:
- name: busybox
  image: busybox
  commands:
  - env
  - ping -c 5 www.baidu.com
```

### 最后
到这里 Drone 这个轮子就算完成了。
<br/>
