---
title: 在Heroku上部署蝉游记
layout: post
date: 2018/05/27 20:40:02
tags : Python
---

### Heroku
[Heroku](https://www.heroku.com/home) 是一个云服务器部署平台，支持多种语言和框架，支持从 Github 自动部署，提供免费的空间和二级域名，虽然系统半小时不使用会自动休眠，但是对我们来说足够用了。

### 教学
1）创建 Heroku 账号。
2）创建一个新的 Heroku App。
3）链接 Github，开启自动部署。
Deploy -> Deployment method -> Github -> Choose Repository & Branch -> Enable Automatic Deploys。
4）在 Github 项目根目录添加 Heroku 配置文件。
* requirements.txt
这个文件里放的是当前项目所有的依赖库和对应版本，我们使用 `pip freeze > requirements.txt` 命令生成。
Ps：
在 Heroku 里，官方使用 [Gunicorn](https://devcenter.heroku.com/articles/python-gunicorn) 来启动 Web Server，所以我们需要在 requirements.txt 里增加一行配置：
```text
gunicorn==19.9.0
```
* Procfile
这个文件是要告诉 Heroku 如何启动这个 Web App。由于我们的 application 文件在二级目录下，所以需要增加 pythonpath 参数。
```text
web: gunicorn --pythonpath ./web/ application:app
```
* runtime.txt（可选）
可以在这个文件中指定你的 python 版本，如果你不想指定，这个文件可以忽略。
```text
python-2.7.10
```
5）提交代码到 Github 之后，返回 Heroku Dashboard。在右侧可以看到项目已经开始自动编译了，等待部署完成，点击右上角 Open App，新世界的大门就此打开。
<br/>
Heroku 项目文件总大小限制要小于 500M（也是合理的，图片应该放在资源服务器上），我自己的游记数据已经接近 3G 自然是放不下了，只好借蝉小队的数据来用用，美其名曰：[你们的回忆我来帮你们保管](https://chanyouji.herokuapp.com/)。

### 写在最后
至此，蝉游记系列到此就结束了，项目简单，也算为自己的情怀做了一些事情 :p
<br/>
