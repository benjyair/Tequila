---
title: Log 筛选小工具
layout: post
date: 2018/06/07 17:02:44
tags : C
---
事情的起因是在和手机厂商合作的过程中，对方测试抓来的全部都是未经筛选的 Main Log，而在 Mac 上我一直没有找到一个能很方便筛选 Log 的工具，类似 Win 上面的 Notepad++。Mac 上我用过 Sublime、Atom、UltraEdit 等等，都不太顺手。后来看了《鸟哥的 Linux 私房菜》，觉得如果只是做一个简单的筛选，用 Shell 应该几行代码就可以完成功能，于是撸了下面一个脚本。脚本比较简单，用着却方便了很多。

```shell
#! /bin/bash
# Shell:
#      logx
# Program:
#      This is a script that filters out strings from file.
# History:
#      2017/06/28 Benjy First release

TMP=${1%}_tmp.txt

cat $1 | while read line
do
	[[ $line == *$2* ]] && echo $line >> $TMP
done
```

用法：
```shell
logx ./log.txt main
```

学了 C 之后，一时找不到练手的项目，就又拿这个需求练手了。

```c
#include <stdio.h>
#include <string.h>
#include <zconf.h>

#define MAX_LINE 1024
#define DEFAULT_SIZE 100

int main(int argc, char *argv[]) {
    char name[DEFAULT_SIZE] = {0};
    char data[DEFAULT_SIZE] = {0};

    if (argc == 3) {
        strcpy(name, argv[1]);
        strcpy(data, argv[2]);
    } else {
        printf("Please input the file name to match: ");
        scanf("%s", name);
        printf("Please input the string to match: ");
        scanf("%s", data);
    }

    char path_in[DEFAULT_SIZE] = {0};
    char path_out[DEFAULT_SIZE] = {0};
    getcwd(path_in, DEFAULT_SIZE);
    getcwd(path_out, DEFAULT_SIZE);
    strcat(path_in, "/");
    strcat(path_out, "/");
    strcat(path_in, name);
    strcat(path_out, name);
    strcat(path_out, ".tmp.txt");

    FILE *fin = NULL;
    FILE *fout = NULL;
    if ((fin = fopen(path_in, "r")) == NULL) {
        printf("Open input file error!\n");
        return -1;
    }
    if ((fout = fopen(path_out, "w")) == NULL) {
        printf("Open output file error!\n");
        return -1;
    }
    printf("Doing\n");
    while (!feof(fin)) {
        char line[MAX_LINE];
        fgets(line, MAX_LINE, fin);
        if (strstr(line, data) != NULL) {
            fprintf(fout, line);
        }
    }
    fclose(fin);
    fclose(fout);
    printf("Done. \n");

}
```

<br/>
