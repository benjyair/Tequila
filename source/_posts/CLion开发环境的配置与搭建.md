---
title: CLion 开发环境的配置与搭建
layout: post
date: 2018/05/31 17:40:11
tags : C
---
从 Android Studio 到 IntelliJ Idea 再到 PyCharm， 已经成了 Jetbrains 的超级粉丝。所以开发 C，果断选择 CLion。而 CLion 中使用 CMake 构建 C 项目。

### C 语言项目的目录结构
以下例子基本涵盖了一个 C 语言大型项目所能用到的所有目录了：
```text
.                               # 项目根目录
|____CMakeLists.txt             # CMake 主配置文件
|____test                       # 测试用例目录
| |____CMakeLists.txt           # 测试模块配置文件
| |____test.c
|____out                        # 输出目录
|____include                    # 头文件 *.h 目录
| |____util                     # header 子目录
| |____xxx
|____lib                        # lib 放置目录
|____build                      # 构建目录
|____src                        # 源码 *.c 目录
| |____CMakeLists.txt           # 源码模块配置文件
| |____util                     # 源码子目录
| |____xxx
```

### 使用 CMake 构建项目
在 CLion 中如果没有配置好 CMake，CLion 的大部分功能都用不了，所以使用 CLion 做开发，配置 CMake 是关键。

<br/>
CMake 主配置文件
```text
cmake_minimum_required(VERSION 3.9)

# 项目名
project(XXX)

set(CMAKE_C_STANDARD 99)

# 把 src 作为子 module
add_subdirectory(src)
```

源码模块配置文件
```text
# PROJECT_SOURCE_DIR 是 CMake 定义的宏，指向项目根目录
include_directories("${PROJECT_SOURCE_DIR}/include/util")    # Util *.h includes

# 把 util 子目录编译成 library，最终会生成 libutil.a 文件
aux_source_directory(./util UTIL_SRC)
add_library(util ${UTIL_SRC})
```

测试用例目录
```text
include_directories("${PROJECT_SOURCE_DIR}/include/util")    # Util *.h includes

# 生成可执行文件 test
add_executable(test test.c)
# 添加链接库 util 等
target_link_libraries(test util)
```
配置完成后，重新 Load 一下项目，在 Run Configurations 中便能看到我们的 test 程序了。 无论是代码跳转还是 Run 和 Debug 就也都能使用了。开发 Android 项目的感觉全都回来了。

<br/>
当然，也可以在命令行中使用如下操作来编译：
```text
cd build
# 在 build 目录下编译整个项目是一个好习惯，否则所产生的 Makefile 文件会充斥在你的源代码中
cmake ../
make
```

<br/>
