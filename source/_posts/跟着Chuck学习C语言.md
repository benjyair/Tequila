---
title: 跟着Chuck学习C语言
layout: post
date: 2018/05/28 19:55:10
tags : C
---
Chuck 说“写 C 很简单，但是想写好 C 很难。”

Chuck 是我司从事嵌入式开发的一枚工程师，器大活好，哦不，人好话少技术吊。

公司有个项目叫 **Braavos**，是由我完成的 Java 版本，现如今需要应用的 Linux 上，就需要开发对应的 C 版本，我毅然决然的接收了这个挑战。别人都是从低级语言向高级语言学习，我这刚好反过来了。所以别人都是问“回调函数在 Java 中如何实现？”，而我则整天感叹“这不就是 Java 里面的接口么！”。

在 [RUNOOB](http://www.runoob.com/cprogramming/c-tutorial.html) 上简单回顾了大学学习的 C 语法，便开始了项目的开发。事实证明，只学习了这些就写大型项目简直是受虐...
所以便有了我向 Chuck 取经的这个过程。

### 函数指针
函数指针的定义
```c
int (*m)(int a, int b);
```
函数指针最大的特点就是：函数指针的函数名是一个指针。

接下来来使用这个函数指针：
```c
int max(int a, int b) {
    printf("The max value is %d \n", a > b ? a : b);
    return 0;
}

int min(int a, int b) {
    printf("The min value is %d \n", a < b ? a : b);
    return 0;
}

int (*m)(int a, int b); // 声明时形参可写可不写

int main()
{
    m = max;
    (*m)(1, 2);

    m = min;
    (*m)(1, 2);
    return 0;
}
```
这不就是 Java 里面的接口么，这不就是对功能的抽象么！

<br/>
