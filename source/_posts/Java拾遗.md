---
title: Java 拾遗
layout: post
date: 2018/06/21 21:40:39
tags : Java
---

### 使用 Try-With-Resources
写了这么多年的 Java，应该如何优雅的开关流？ Try-Finally？看看下面的写法：
```java
public static void main(String args[]) {
    try (InputStream is = new InputStream(/* resources */)) {
        // Do something.
    }
}
```
这种写法相比在 Finally 里面来关闭流，或者手动维护流来说能减少很多代码，IO 会在 Try 执行完毕后自动被关闭，代码看起来也好看一些。深入探究原理，其实只要是实现了 AutoCloseable 接口的类都可以使用这种方法来做到自动关闭，这样可以应用的场景就多了很多。
<br/>

### 使用 Lable
假如在多层循环里，我们需要跳转到某一层循环中，或者说跳出整个循环继续执行后面的逻辑，该怎么做？在 C 中使用 goto 可以很轻易的做到，在 Java 中看起来就不那么容易完成了，因为 Java 中没有 goto。Java 中控制流程的关键字有三个，continue、break 和 return。我们其实可以使用 continue 或 break 配合 lable 来达到 goto 的效果。如下面的代码示例：
```java
public static void main(String args[]) {
    int[] array = {1, 2, 3};
    System.out.println("Start");
    done:
    for (int i : array) {
        System.out.println(String.format("i = %d", i));
        for (int j : array) {
            System.out.println(String.format("j = %d", j));
            if (i == 2 && j == 2) {
                break done; /* 这里直接 break 到最外层循环 */
            }
        }
    }
    System.out.println("Finish");
}
```
<br/>
