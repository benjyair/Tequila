---
title: Android 系统对 eSIM 的支持
layout: post
date: 2018/10/21 19:07:32
tags : eSIM
---
从 Android 8.1 开始，Google 开始提供全新的 eUICC API，主要实现了 GSMA SGP.ES10x 接口。并封停之前 Carrier Privileges 对 ISD-R 的操作 ( *Only allows LPA to open logical channel to ISD-R ------ 引用自 Android 8.1 源码* )。自此以后，基本封死了第三方开发 LPA（*Local Profile Assistant*）的可能性，至于 LPA 如何集成，控制权交给了 OEM 厂商。

<br/>

![LPA](https://source.android.com/devices/tech/connect/images/carrier-oem-lpa.png)
上图中：
* **EuiccManager**
应用程序与 LPA 交互的入口。包括可下载、删除及切换到运营商所拥有的资源。此外，这还包括 LUI（*Local User Interface*）系统应用。
* **EuiccController**
EuiccManager 与 EuiccService 之间的桥梁，负责消息的分发和回调。
* **EuiccService/EuiccServiceImpl**
LPA 后端服务的提供者，也是可以称之为 LPD（*Local Profile Download*）。EuiccService 是一个接口，定义了 ES10x 的接口规范，LPA 通过实现 EuiccService 来响应 EuiccController 的调用。
* **EuiccCardController**
EuiccCardManager 是用于与 eSIM 芯片进行通信的接口。负责处理低级 APDU 的请求/响应以及 ASN.1 的解析。

具体使用方法可以参照官网介绍：[点击这里](https://source.android.com/devices/tech/connect/esim-overview#making_a_carrier_app)

<br/>
最后说一下个人的看法:
1、eSIM 在终端上的趋势约来约明显了，Android 和 IOS 相继都推出了各自的 eSIM Demo。
2、更加全面统一了，GSMA 统一了规范，平台统一了框架，OEM 统一了入口，开发者之后调用相应的 API 也会更加方便。
3、安全性方面更加严格了，第三方很难插足。
4、Google 你只向下实现了 ES10x 的接口，那 LPA 中向上对 SMDP+ 的 ES9+ 接口怎么吧？？？

<br/>
以上。
<br/>
