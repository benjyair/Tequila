---
title: 在树莓派上搭建 ownCloud 私人网盘
layout: post
date: 2018/07/28 16:11:12
tags : 树莓派
---
两年前买了一台树莓派，放在手里已经吃灰好久了，最近玩 C 语言又拿了出来，主要用它来编译一下项目（Mac OS 不区分大小写，引用的某些库在 Mac 下是无法编译），不过两台机器之间文件互传也是很麻烦。后来在树莓派上折腾 NAS ([ownCloud](https://owncloud.org/)) 的时候忽然发现了一条新的路子，把 Mac 上的工作目录放在 NAS 下，修改的代码自动同步到树莓派上，然后 SSH 过去编译，编译完成后生成的库文件也会自动同步回我的 Mac 上，完美。
<br/>
我的树莓派是 Debian 系统，本篇文章也是针对 Debian 的教程。
### 更新树莓派
```shell
sudo apt update && apt upgrade
```

### 安装 Apache 并启动
```shell
sudo apt install apache2

systemctl start apache2
systemctl enable apache2
```

### 安装 Mysql
安装数据库，并以 Root 用户登录。
```shell
sudo apt install mysql-server

mysql -u root -p
```

为 ownCloud 创建数据库和用户。
```sql
MariaDB [(none)]> create database owncloud;
 Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create user owncloud@localhost identified by '12345';
 Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all privileges on owncloud.* to owncloud@localhost identified by '12345';
 Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
 Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit;
 Bye
 ```

### 安装 PHP 及依赖组件
```shell
sudo apt install -y apache2 mariadb-server libapache2-mod-php7.0 \
    php7.0-gd php7.0-json php7.0-mysql php7.0-curl \
    php7.0-intl php7.0-mcrypt php-imagick \
    php7.0-zip php7.0-xml php7.0-mbstring
```

### 安装 ownCloud 并部署
```shell
cd ~
wget https://download.owncloud.org/community/owncloud-10.0.10.tar.bz2
tar -xvf owncloud-10.0.10.tar.bz2
chown -R www-data:www-data owncloud
mv owncloud /var/www/html/
```
### 配置 Apache 服务器
创建一个新配置文件。
```shell
sudo nano /etc/apache2/sites-available/owncloud.conf
```
复制以下内容进去并保存。
```text
Alias /owncloud "/var/www/html/owncloud/"

<Directory /var/www/html/owncloud/>
 Options +FollowSymlinks
 AllowOverride All

<IfModule mod_dav.c>
 Dav off
 </IfModule>

SetEnv HOME /var/www/html/owncloud
SetEnv HTTP_HOME /var/www/html/owncloud

</Directory>
```
创建链接文件。
```shell
sudo ln -s /etc/apache2/sites-available/owncloud.conf /etc/apache2/sites-enabled/owncloud.conf
```
以下是可选操作。
```shell
a2enmod headers
systemctl restart apache2
a2enmod env
a2enmod dir
a2enmod mime
```

### 通过浏览器配置 ownCloud
使用树莓派上的浏览器访问 `http://localhost/owncloud`，依次输入：
Username: owncloud
Password: 12345
Database: owncloud
Server: localhost
Data folder 使用默认的地址即可（也可以挂载到外置硬盘）。
等待设置完成，免费的私有云便搭建好了。
ownCloud 还提供不同平台的客户端，[下载地址](https://owncloud.org/install/#install-clients)。

### 配置内网访问
刚设置好的 ownCloud 只能本机使用 `localhost` 来访问，如果要配置内网、外网的访问，就必须把对应的 IP 、域名加入到安全域才行。
```shell
sudo nano /var/www/owncloud/config/config.php
```
在 `trusted_domains` 里面增加一条新的记录，比如我的树莓派的内网 IP 是 `172.16.20.68` ，那么修改后的内容如下：
```test
 'trusted_domains' =>
  array (
    0 => 'localhost',
    1 => '172.16.20.68',
  ),
```
试一下在 Mac 上访问 `http://172.16.20.68/owncloud`。

以上。
<br/>
