---
title: linux指令-find
date: 2020-12-30 21:20:42
tags: Linux
categories: Linux
---
# find 命令

find命令主要用于文件搜索，那是相当的快呀。

**find命令使用格式：**

```linux
find path test action
```

+ path表示要搜索的目录，可以同时搜索多个目录，用空格隔开
+ test表示条件，可以有多个条件
+ action为搜索后的动作

## 条件（test）

| 条件            | 说明                                           |
| --------------- | ---------------------------------------------- |
| -name pattern   | 通过文件名进行查找                             |
| -type f/d       | 通过文件类型进行查找，d=directory，f=file      |
| -perm pattern   | 按文件或者文件夹的权限进行匹配（例如644、755） |
| -user userId    | 查找文件为指定用户id的文件                     |
| -group groupId  | 查找组为指定用户组的文件                       |
| -size [+/-]size | 按文件大小进行查找                             |
| -empty          | 查找空文件                                     |
| -amin [+/-]min  | 查找最后访问时间小于/大于指定分钟的文件        |
| -atime [+/-]day | 查找最后访问时间小于/大于指定天数的文件        |
| -cmin [+/-]min  | 查找最后改变时间小于/大于指定分钟的文件        |
| -ctime [+/-]day | 查找最后改变时间小于/大于指定天数的文件        |
| -mmin [+/-]min  | 查找最后修改时间小于/大于指定分钟的文件        |
| -mtime [+/-]day | 查找最后修改时间小于/大于指定天数的文件        |

其中要对size进行一下重点说明，在size后指定的大小，根据man文档，共有以下几种单位

+ b (521-byte blocks)
+ c (bytes)
+ w (two-bytes)
+ k (Kilobytes)
+ M (Megabytes)
+ G (Gigabytes)

> size 是通过 + 或者-来标记大于或者小于

```bash
# 例子
find ./ -name *.sh -o -name *.txt

find ./ -perm 644 -exec ls {} \;

find * -user 0

find  * -size +2c

find  * -size -2k
```

## 动作（action）

| 参数                | 解释                               |
| ------------------- | ---------------------------------- |
| -print（默认）      | 将结果输出到标准输出               |
| -fprint filename    | 将结果输出到指定文件               |
| -ls                 | 打印出查找到的文件详情             |
| -fls filename       | 将查找到的文件已详细方式输出到文件 |
| -delete             | 将查找到的文件删除                 |
| -exec command {} \; | 查找到文件并且执行命令             |
| -ok command  {} \;  | 查找到文件并执行相应的命令         |

前面几个都能知其意，关于-exec command {} \;，意思是将查找到的文件再进行命令的处理，其中{}为查找到的所有文件，\ 表示-exec命令的结束，而且要带有分号。

> 注意：在{}左右两侧都有空格

那么exec和ok又什么区别呢，exec在执行后面的命令时不需要用户再次确认，而ok则需要用户进行再次的确认。

```bash
[root@centos test]# find ./ -name "a*" -ok ls {} \;
< ls ... ./aaaa > ? yes
./aaaa
[root@centos test]# find ./ -name "a*" -exec ls {} \;
./aaaa
```

## 逻辑运算

在find中也存在逻辑关系

**或是用 -o参数代替的**，记住，它是从后面计算的，例如

```bash
find ./ -name "a*" -or  -name "b*"  -exec cat {} \+;
```

他会先从b*这个条件开始判断，满足这个条件就不往后走了，最后只输出了b\*文件中的内容

**与关系是直接使用空格即可**

```bash
find ./ -name "c*" -size +2k
```

**非关系是使用的！**

```bash
# 查找非c开头的文件
find ./ ! -name "c*"
```

## 特殊情况

当find命令查找持续了几分钟找到很多文件时，这时-exec后的命令可能就会执行出错（因为有的命令接收的文本长度是有限制的），那么我们就可以使用xargs来解决问题。

xargs可以帮助我们批量的读出数据，并不是一起读出来，读出来的还是文件的内容（不是文件名）

```bash
find ./ -type f -name "*.sh" -o -name "*.txt" | xargs grep -i "hello world"
```

## 精彩例子

```bash
#在当前目录下查找所有用户具有读、写和执行权限的文件，并收回相应的写权限。
find . -perm -644 -print | xargs chmod o-w  
   
#查找所有的jpg 文件，并且压缩它。
find / -name *.jpg -type f -print | xargs tar -cvzf images.tar.gz

#尝试用rm删除太多文件，你可能得到错误信息：/bin/rm Argument list too long。用xargs去避免这个问题。
find ~ -name ‘*.log’ -print0 | xargs -i -0 rm -f {}

#拷贝所有的图片文件到一个外部的硬盘驱动。
ls *.jpg | xargs -n1 -i cp {} /external-hard-drive/directory
```

统计一个目录下文件个子目录的个数

```bash
#!/bin/bash

fileCount=`find $1 -type f -print | wc -l`
dicCount=`find $1 -type d -print | wc -l`

echo "$1 has $fileCount files"
echo "$1 has $dicCount directories"
```

## 参考链接

+ https://www.jianshu.com/p/2eb1ab12c9ce
