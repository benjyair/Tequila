---
title: 从 Framework 层看 Android 启动
layout: post
date: 2014/12/04 13:01:13
tags : 读书笔记
---

众所周知，Linux 中所有的进程都是有 init 进程创建并运行的，Android 系统基于 Linux 内核，所以也存在着 init 进程。
init进程启动的过程比较复杂，但是在准备工作做好之后，会通过 jni 创建 Dalvik 虚拟机，然后启动 Android 的核心 Zygote 进程，这个进程将成为所有应用进程的孵化器存在。下面我通过几个图来说明这个过程。

### Init 进程启动以及 Zygote 启动

![tool-editor](https://blog-1251733178.cos.ap-beijing.myqcloud.com/20141204181105.jpg)

上图是启动 Android 设备之后的进程，可以明显看到在 init 进程启动之后启动了 Zygote 进程，而 Zygote 作为之后的大部分进程的父进程存在。

### Zygote 启动应用程序

![tool-editor](https://blog-1251733178.cos.ap-beijing.myqcloud.com/20141204181056.jpg)

上图是 Zygote 进程启动应用程序的流程图，Zygote 进程调用 fork（）函数创建出 Zygote 子进程，子进程共享父进程的代码区和链接信息，但是注意，新的 Android 应用程序并非通过 fork（）来重新装载已有的进程代码区，而是动态的加载到复制出的Dalvik虚拟机上，而后，Zygote 进程将执行流程交给应用程序，Android 应用程序开始运行，新生的应用程序拥有 Zygote 的进程库和资源的链接信息，所以运行速度很快。

### Android Framework 的启动过程

![tool-editor](https://blog-1251733178.cos.ap-beijing.myqcloud.com/20141204181100.jpg)

上图是 Android Framework 的启动过程，Zygote 启动 Dalvik 虚拟机后，会在生成一个 Dalvik 虚拟机示例，以便运行名称为 SystemServer 的 Java 服务，SystemServer 用于运行 Audio Flinger 与 Surface Flinger 本地服务，运行完本地服务之后开始运行 Android Framework 的 Java 服务，也就是我们在 Android 系统架构图中 Application Framework 中的各种 Manager Server
下图我们从代码层看 Android Framework 的启动过程

![tool-editor](https://blog-1251733178.cos.ap-beijing.myqcloud.com/20141204181052.jpg)

### Binder IPC 机制

![tool-editor](https://blog-1251733178.cos.ap-beijing.myqcloud.com/20141204181047.jpg)

由于 Android 应用程序与系统服务不在同一个系统进程中，这里就引入的 Binder IPC 机制，服务使用者调用 foo（）服务代理函数，而后 foo（）服务代理函数通过 Binder RPC 调用 Foo 服务的 foo（）服务 Stub 函数

首先服务使用者调用 foo（）代理函数，传递 Binder RPC 数据，该数据包含引用 Foo 服务的请求。Binder RPC 数据经过 Marshalling 处理后，由 Service Framework 生成 Binder IPC 数据，然后通过 Binder Driver 传递给 Service Server 端

Service Server 端收到 Binder IPC 数据后，由 Service Framework 对数据进行 UnMarshalling 处理，然后传递给 Service Stub的onTransact（）函数，Service Stub 根据 Binder IPC 数据中的 RPC 代码判断它是一个针对 Foo服务的foo（）服务 Stub 函数的 Binder RPC，最后， 以 Binder IPC 数据中包含的 Binder RPC 数据为参数，调用 foo（）服务的 Stub 函数。至此，就是 Binder IPC 的整个流程。

————摘自《Android框架揭秘》 以上部分是这本书中的大部分知识的总结，其中的这几幅图更是这些重点简单明了的描述。当然 Binder 机制只看图还是不能深入了解的，因为内容比较多，所以还是推荐看书，写的不错。