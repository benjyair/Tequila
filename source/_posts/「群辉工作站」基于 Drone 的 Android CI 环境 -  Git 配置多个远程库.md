---
title: 「群辉工作站」基于 Drone 的 Android CI 环境 -  Git 配置多个远程库
layout: post
date: 2021/03/24 12:22:12
tags : Docker
---

### 设置 Git 备用库
公司的代码现在是在 Github 或者其他平台上的，而我们搭建的 CI 平台只是自己的一个私有系统，且依赖于我们自己搭建的 Gitea，那么如何方便的让两个仓库完成同步呢？比如项目的源码，主提交至 Github，同时在 Gitea 中保存一个同步备份。Push 时同时提交到两个仓库，Pull 时从 Github 仓库获取。
这个问题很简单，一行命令就能解决。
首先正常 Clone 下我们原本的项目然后执行以下命令添加备用库：
```shell
git remote set-url --add origin http://192.168.192.1:3000/benjyair/Draque.git
```
以上，就能达到我们的目的了。
