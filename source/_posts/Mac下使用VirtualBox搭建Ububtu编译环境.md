---
title: Mac 下使用 VirtualBox 搭建 Ububtu 编译环境
layout: post
date: 2018/06/03 13:49:50
tags : Linux
---

由于 Mac 系统默认是大小写不敏感的，所以在编译 C 的时候，如果项目中出现文件名一样，大小写不一样的两个文件时，在 Mac 下编译就会出现问题。比如： **ABC.c** 和 **abc.c** ，这种情况只能换到 Linux 下编译。而我又想在 Mac 下完成代码开发，那使用 VirtualBox 搭建 Ububtu 的编译环境 + 共享文件夹共享源代码就是最好的选择。不要和我说使用 Mac + Linux 服务器的方式，我试过了，体验很差。

### 安装和配置虚拟机
* 下载 VirtualBox 和 Ububtu iso，按提示一步一步来就能很轻松完成 Ubuntu 的安装。
* 设置 root 密码，为后续简化使用共享文件夹，我们将直接使用 root 账户登录 Ububtu。
```text
sudo passwd
```
* 安装 openssh 服务
```text
sudo apt-get install openssh-server
```
* 修改 ssh 配置
```text
nano /etc/ssh/sshd_config
# 找到 Port 22 取消注释开启 22 端口
Port 22
# 找到 PermitRootLogin without-password 修改为 yes 允许 root 账户使用 ssh 登录
PermitRootLogin yes
```
* 重启 ssh 服务
```text
sudo /etc/init.d/ssh restart
```
* 设置虚拟机端口转发
```text
# 在 VirtualBox 中找到配置表 VirtualBox -> Settings -> Network -> Adapter 1 -> Advanced -> Port Forwarding
# 添加一个 Item
| Name   | Protocol | Host IP | Host Port | Guest IP | Guest Port |
| Rule 1 | TCP      |         | 1111      |          | 22         |
```
* 安装 VirtualBox Guest Additions
```text
VirtualBox VM -> Devices -> Insert Guest Additions CD image...
cd /media/
./VBoxLinuxAdditions.run
reboot
```
* 设置共享文件夹
```text
在 VirtualBox -> Settings -> Share Folders 添加共享文件夹，勾选 Auto-mount 和 Make Permanent
```
* 使用 ssh 远程登录虚拟机
```text
ssh -p 1111 root@127.0.0.1
ls /media/
```
到这里整个环境就设置好了，接下来便可以在 Mac 下开发 C 项目，在 Terminal 中编译项目了。

<br/>
