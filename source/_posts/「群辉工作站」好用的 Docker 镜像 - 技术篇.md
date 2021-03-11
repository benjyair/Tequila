---
title: 「群辉工作站」好用的 Docker 镜像 - 技术篇
layout: post
date: 2021/03/02 22:19:34
tags : Docker
---

### Baota
宝塔 Linux 面板，快速建站，运维管理平台，这个真的很方便，尤其是网站备份功能。

[宝塔面板一键docker部署](https://registry.hub.docker.com/r/pch18/baota/)

```yaml
version: '3'

services:
  baota:
    image: pch18/baota:lnmp
    container_name: baota
    restart: always
    volumes:
      - ./www/backup:/www/backup
      - ./www/wwwroot:/www/wwwroot
    ports:
      - "4020:20"
      - "4080:2080"
      - "4021:21"
      - "4443:443"
      - "4888:888"
      - "4000:8888"
```
通过 `http://192.168.192.1:4000` 访问，如遇密码错误需要先到容器 shell 中通过 `bt` 命令修改用户名和密码。

### Portainer
[Portainer](https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/) 是一个非常强大的 Docker 容器图形化管理平台。本文使用 `ce + agent` 模式。

```yaml
version: '3'

services:
  portainer-service:
    image: portainer/portainer-ce:latest
    container_name: portainer-service
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data:/data
    ports:
      - "7000:9000"
    environment:
      - TZ=Asia/Shanghai
      - DOCKER_API_VERSION=1.39
    command: -H tcp://192.168.192.1:7001 --tlsskipverify
  portainer-agent:
    image: portainer/agent:latest
    container_name: portainer-agent
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./volumes:/var/lib/docker/volumes
    ports:
      - "7001:9001"
    environment:
      - TZ=Asia/Shanghai
      - DOCKER_API_VERSION=1.39

```

通过 `http://192.168.192.1:7000` 访问。

### MinIO
[MinIO](https://docs.min.io/cn/) 是一个非常轻量的对象存储服务，用来做文件服务器。
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

通过 `http://192.168.192.1:9090` 访问。
```text
Object API (Amazon S3 compatible):
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
```

### Jupyter
Jupyter 服务。
需要自行创建挂载的 data 目录，且增加其他组写权限 `drwxr-xr-x+` --> `drwxrwxrwx`，原因是容器用户 `jovyan` 需要向该目录写入数据。
```yaml
version: '3'

services:
  jupyter:
    image: jupyter/datascience-notebook:latest
    container_name: jupyter
    restart: always
    volumes:
      - ./data:/home/jovyan/work
    ports:
      - "8088:8888"
```

通过 `http://192.168.192.1:8088` 访问，登录 Token 在日志中获取。

### aliyun-ddns-cli
阿里云域名 dns 解析，需要公网 IP。
```yaml
version: '3'

services:
  aliyun-ddns:
    image: chenhw2/aliyun-ddns-cli:latest
    container_name: aliyun-ddns
    restart: always
    environment:
      - AKID=LTHB4GD6
      - AKSCT=Vn54P8UQwxw0
      - DOMAIN=nas.benjyair.com
      - REDO=600
```
