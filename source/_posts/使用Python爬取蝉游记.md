---
title: 使用Python抓取蝉游记
layout: post
date: 2018/05/21 21:40:39
tags : Python
---

### 项目起因
银叔的蝉游记从入坑以来一直非常喜欢，画卷一样的布局、旅行中美好的回忆以及满世界打卡的功能深深吸引着我。但自从蝉游记易手之后，项目能否继续活下去一直都存在疑问。2017 年底，蝉游记服务器出现过多次 404 和 500，由此萌生了离线蝉游记的这个想法。项目名 [Taki](https://github.com/benjyair/Taki) 取自新海诚《你的名字》中 "泷" 的名字，希望借电影的寓意来保存自己在蝉游记中的一些回忆。
<br/>
项目目前计划分为三个部分：
* 爬虫部分，抓取蝉游记的数据保存在本地。
* 服务器部分，使用本地数据尽可能的还原蝉游记原貌。
* (TBD) 生成静态网页。

### 使用 Charles 抓取 API、Header 以及 Cookie
API：
蝉游记虽然作为一款几年前的产品，不得不说系统还是设计的非常不错的(因为 ruby on rails ？)，良好的 RESTful API 对我们来说非常友好。
* http://chanyouji.com/api/users/1.json?page=1 用户信息及游记列表
* http://chanyouji.com/api/trips/310965.json   游记详情
* http://chanyouji.com/api/users/map/1.json 旅行地图

Header：
```python
headers = {'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8',
               'Accept-Encoding': 'gzip, deflate',
               'Accept-Language': 'zh-CN,zh;q=0.9',
               'Cache-Control': 'no-cache',
               'Connection': 'keep-alive',
               'Host': 'chanyouji.com',
               'Pragma': 'no-cache',
               'Upgrade-Insecure-Requests': '1',
               'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36'
       }
```

Cookie：
抓到的 cookie 是无法持续使用的，我们这里采用动态的方式，先请求蝉游记首页，然后记录下来留给后面的请求使用。
```python
import requests

session = requests.session()
response = session.get('http://chanyouji.com')
cookies = response.cookies.get_dict()
```

### 数据抓取
由于有 API 的缘故，我们数据抓取的工作量至少减少了 80% ，这里就不过多赘述。我们采用 requests 库来做网络请求，简单起见，没有使用多线程也没有使用代理，所以为了防止多次请求被拉黑，每次请求完随机 Sleep 几秒。

### 数据存储
数据抓到了，接下来便是数据的存储。除了 JSON 数据，还需要把图片也一起存下来，毕竟图片是游记的基础。由于已经拿到了 JSON，这里我就不使用数据库了，使用文件按下面的目录结构进保存：
```text
|____data                       # 缓存目录
| |____user_1                   # 用户数据目录
| | |____1.json                 # 用户 JSON 数据
| | |____cover_photo_xxx.jpg    # 游记封面照片
| | |____image_xxx.jpg          # 用户头像照片
| | |____map.json               # 用户旅行 JSON 数据
| | |____trip_47891             # 游记目录
| | | |____xxxxxxx.jpg          # 游记照片
| | | |____310965.json          # 游记 JSON 数据
```

游记照片 URL 抽取，trips 的数据结构大致结构如下：
```json
{"trip_days":[{
    "nodes":[{
        "notes":[{
            "photo":[{
                "url":""
            }]
        }]
    }]
}]}
```
如此多层次的嵌套，一层一层解析下去代码少说十几行。这里我使用 Python 的列表生成式只用一行代码完成 URL 的筛选，入参 days 即 trip_days。
```python
def parse_image_url(days):
    return [note['photo']['url'] for day in days for node in day['nodes'] for note in node['notes'] if note.get('photo')]
```

[项目地址](https://github.com/benjyair/Taki)
<br/>
