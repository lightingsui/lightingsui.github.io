---
title: linux指令-cat-tac
date: 2021-02-19 10:44:26
tags: cat-tac
categories: linux
---
# linux指令-cat-tac

cat指令以及tac指令用来查看文件的内容，使用也是十分便利

**命令格式：**

```bash
# cat
cat [OPTION]... [FILE]...

# tac
tac [OPTION]... [FILE]...
```

**参数：**

| cat参数 | 解释                                       |
| ------- | ------------------------------------------ |
| -b      | 列出行号，只列出非空白行，空白行不进行编号 |
| -n      | 列出行号，空白行也进行编号                 |
| -E      | 将结尾的断行字符$打印出来                  |
| -T      | 将tab以^I打印出来                          |
| -v      | 列出一些看不出来的特殊字符                 |
| -A      | 相当于-vET的整合，列出特殊字符             |

**演示：**

查看`/etc/man_db.conf`

```bash
[root@VM-8-14-centos ~]# cat /etc/man_db.conf
#
#
# This file is used by the man-db package to configure the man and cat paths.
# It is also used to provide a manpath for those without one by examining
# their PATH environment variable. For details see the manpath(5) man page.
# 以下部分省略
```

显示行号（-n与-b的区别）

```bash
[root@VM-8-14-centos ~]# cat -b /etc/issue
     1  \S
     2  Kernel \r on an \m

[root@VM-8-14-centos ~]# cat -n /etc/issue
     1  \S
     2  Kernel \r on an \m
     3
```

列出所有特殊字符

```bash
[root@VM-8-14-centos ~]# cat -A /etc/man_db.conf
# $
#$
# This file is used by the man-db package to configure the man and cat paths.$
# It is also used to provide a manpath for those without one by examining$
# their PATH environment variable. For details see the manpath(5) man page.$
#$
# Lines beginning with `#' are comments and are ignored. Any combination of$
# tabs or spaces may be used as `whitespace' separators.$
#$
# There are three mappings allowed in this file:$
# 以下部分省略
```

在windows和linux中，断行符号是不一样的，在linux中，断行符号为$，而在windows中，断行符号为^M$。我们在windows中写一个txt文件，然后发送到linux中查看，看一下断行符的样子

```bash
[root@VM-8-14-centos ~]# cat -A windows.txt
aaaaaaaaaaaaaaaaaaaaaa^M$
bbbbbbbbbbbbbbbbbb^M$
```

行数较少的文件，使用cat查看即可，一旦文件的行数较多，使用cat查看是不便利的，应当使用less、more等命令查看

至于tac命令，就是将文件以相反的顺序输出，就犹如cat与tac字母位置的区别

```bash
[root@VM-8-14-centos ~]# cat /etc/issue
\S
Kernel \r on an \m

[root@VM-8-14-centos ~]# tac /etc/issue

Kernel \r on an \m
\S
```




