---
title: 「群辉工作站」基于 Drone 的 Android CI 环境 - 搭建 Gitea
layout: post
date: 2021/02/24 20:10:44
tags : Docker
---

### 了解 Gitea
Gitea 与其他 Git 托管工具的横向对比：[点击这里](https://docs.gitea.io/zh-cn/comparison/)

选择 Gitea 有两个原因：

1. 低资源开销，需要能运行在 Nas 上面，我这里使用的是群辉 DS720+。
2. 兼容性好，这次主要目的是与 Drone 配合搭建 CI 平台，Drone 官网中提到 Gitea 与 Drone 的兼容性更好。

### 创建容器
在群辉上搭建 Gitea 和在其他平台上无异，首先安装 docker 工具即可。
具体操作过程可以使用群辉 Docker 套件提供的图形用户界面，或者使用 docker-compose。
使用套件每次都需要重新输入参数不利于调试与迁移，这里就不展开详细讲解了，参考 docker-compose 中的配置完全可以模仿的来。
这里主要是使用 docker-compose 的方式，首先准备`docker-compose.yml`文件：

```yaml
version: "3"

services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: always
    volumes:
      - ./data:/data
    ports:
      - "3000:3000"
      - "3022:22"
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=mysql
      - DB_HOST=192.168.192.1:3306
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=abc123
```
以上配置使用了 Mysql 数据库，使用 Gitea 前需要在 Mysql 中创建好用户与数据库，如需修改也可以在初始化 Gitea 的时候再修改。以下是对应的几个 SQL 指令：

```sql
CREATE USER 'gitea'@'%' IDENTIFIED BY 'abc123';
ALTER USER 'gitea'@'%' IDENTIFIED BY 'abc123' PASSWORD EXPIRE NEVER;
FLUSH PRIVILEGES;
ALTER USER 'gitea'@'%' IDENTIFIED WITH mysql_native_password BY 'abc123';
GRANT ALL ON *.* TO 'gitea'@'%';
create database gitea;
```

SSH 登入群辉准备创建容器，创建过程中会自动下载 Gitea 映像，也可以提前在 Docker 套件注册表中提前搜索下载。

```shell
# 1、在 File Station 中上传 docker-compose.yml 文件到 docker 共享文件夹的 gitea 目录，
#    如果没有就自己创建。我的 docker 共享文件夹在 存储池2 上，所以 compose 文件的路径就是：
#    /volume2/docker/gitea/docker-compose.yml
# 2、ssh 登入群辉，ssh 用户名@群辉IP
ssh benjy@192.168.192.1 
# 3、获取 root 权限，进入 compose 路径
sudo -i  # 再输入一次当前用户密码就行
cd /volume2/docker/gitea/
# 4、创建容器
docker-compose up -d
```
到这里容器就创建好了，去 Docker 套件中检查容器是否创建完成并成功启动，如果后续需要调整参数，需要在套件中删除当前容器后重新执行第四条命令。

### 初始化 Gitea
打开浏览器访问 Gitea 服务：`http://192.168.192.1:3000/`

按指引依次设置数据库和 Gitea，我们在 compose 文件中提前设置好了数据库参数，所以这里已经自动帮我们填写上去了，如有改动也可以在这个时候重新修改。
Gitea 设置中基本不需要修改，SSH 服务域名和基础 URL 记得从 localhost 改成容器主机的 ip，管理员账户可以提前设置，方便后面使用，
然后点击立即安装，这样 Gitea 就初始化完成了。

### 最后
到这里 Gitea 这个轮子就算完成了，后面开始准备搭建 Drone。
<br/>
[Gitea install with docker](https://docs.gitea.io/en-us/install-with-docker/)
<br/>
