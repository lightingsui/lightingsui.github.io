---
title: linux指令-ls
date: 2021-02-05 18:39:39
tags: ls
categories: linux
---
# linux指令-ls

ls命令全称为`list directory contents`，是linux中最最重要的命令之一，通常用来查看目录下的文件，也算是linux入门指令了吧

**命令格式：**

```bash
ls [OPTION]... [FILE]...
```

**参数信息：**

| 参数                       | 解释                                                         |
| -------------------------- | ------------------------------------------------------------ |
| -a                         | 列出全部的信息，**包括**"."和".."                            |
| -A                         | 列出全部信息，**不包括**"."和".."                            |
| -C                         | 按列列出条目                                                 |
| -d                         | 列出目录本身，包括目录中的内容                               |
| -f                         | 直接列出结果，不排序                                         |
| -F                         | 在列出内容的末尾加上了标识，/ 代表目录，= 代表socket文件 l 代表FIFO文件，普通文件不标识 |
| -h                         | 以人类易读的方式展示出文字大小                               |
| -i                         | 列出inode号                                                  |
| -l                         | 列出文件的详细信息                                           |
| -n                         | 基于-l，不展示用户名和用户组名，取而代之的是UID与GID         |
| -r                         | 反向排序结果                                                 |
| -R                         | 递归的列出目录下的所有内容                                   |
| -S                         | 根据文件的大小排序                                           |
| -t                         | 根据文件的**最近访问时间（ctime）**排序                      |
| --color=never              | 不根据文件的类型展示颜色                                     |
| --color=always             | 总是根据文件的类型显示颜色                                   |
| --color=auto               | 让系统自行决定是否根据文件的类型显示颜色                     |
| --full-time                | 显示完整的时间                                               |
| --time={atime,ctime,mtime} | 设置ls -l时展示的时间                                        |

**演示：**

ls默认展示的样式

```bash
[root@VM-8-14-centos test]# ls
aaaa  bbbbbb  cccc  col_test.txt  cut_test  my  my_hard  my_soft  uniq_test
```

`ls -l`字段信息

```bash
[root@VM-8-14-centos test]# ls -l
总用量 36
-rw-r--r-- 1 root root   30 12月 30 18:26 aaaa
-rw-r--r-- 1 root root   16 12月 30 18:26 bbbbbb
-rw-r--r-- 1 root root 2701 12月 30 20:22 cccc
-rw-r--r-- 1 root root   21 2月   5 10:46 col_test.txt
-rw-r--r-- 1 root root   25 2月   5 11:48 cut_test
drwxr-xr-x 2 root root 4096 2月   5 18:07 my
drwxr-xr-x 2 root root 4096 2月   5 18:09 my_hard
drwxr-xr-x 3 root root 4096 2月   5 18:09 my_soft
-rw-r--r-- 1 root root  111 2月   4 14:47 uniq_test
```

```bash
-rw-r--r-- 1 root root   30 12月 30 18:26 aaaa
+ 文件的类型以及权限，- 代表普通文件，d 代表目录，rwx为权限
+ 1代表被硬链接的次数
+ 第一个root为文件所属主
+ 第二个root为文件所属组
+ 30为文件的大小，默认以byte方式展示
+ 12月 30 18:26代表文件修改（mtime）的时间
+ aaaa为文件名称
```

> 另外，在Centos7中bash默认为 `ls -l`提供了别名 `ll`
>
> 还有一点需要说明，如果文件的修改时间是在半年内发生的，那么时间的显示格式为 月 日 小时 分钟，如果是发生的时间超过半年，将展示完整的信息。如果需要查看完整的信息，可以使用`--full-time`选项

展示`ls -lh`

```bash
[root@VM-8-14-centos test]# ls -lh
总用量 36K
-rw-r--r-- 1 root root   30 12月 30 18:26 aaaa
-rw-r--r-- 1 root root   16 12月 30 18:26 bbbbbb
-rw-r--r-- 1 root root 2.7K 12月 30 20:22 cccc
-rw-r--r-- 1 root root   21 2月   5 10:46 col_test.txt
-rw-r--r-- 1 root root   25 2月   5 11:48 cut_test
drwxr-xr-x 2 root root 4.0K 2月   5 18:07 my
drwxr-xr-x 2 root root 4.0K 2月   5 18:09 my_hard
drwxr-xr-x 3 root root 4.0K 2月   5 18:09 my_soft
-rw-r--r-- 1 root root  111 2月   4 14:47 uniq_test
```

文件的大小将变为易读的`K、M、G`的形式

在ubuntu中，存在一个默认的别名`l`

```bash
alias l='ls -CF'
```
