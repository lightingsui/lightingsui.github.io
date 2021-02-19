---
title: linux指令-cp
date: 2021-02-05 20:22:48
tags: cp
categories: linux
---
# linux指令-cp

cp指令我们一般用来进行文件的复制，全称为 copy files and directories。但是，不同的身份执行这个命令会得到不同的结果。

**命令格式：**

```bash
cp [OPTION]... [-T] SOURCE DEST
cp [OPTION]... SOURCE... DIRECTORY
cp [OPTION]... -t DIRECTORY SOURCE...
```

**参数：**

| 参数 | 解释                                                         |
| ---- | ------------------------------------------------------------ |
| -a   | 相当于-dr                                                    |
| -d   | 若复制的文件为软链接文件，则会复制软链接所指的文件，而非软链接本身 |
| -f   | 强制执行                                                     |
| -i   | 若目标文件存在，则询问是否覆盖                               |
| -l   | 对文件进行硬链接                                             |
| -s   | 对文件进行软链接                                             |
| -p   | 连同文件的属性一起copy，备份使用                             |
| -r   | 复制目录时使用                                               |
| -u   | 当源文件比目标文件新的时候更新目标文件，或者目标文件不存在   |

**演示：**

首先touch一个`a.txt`

```bash
[root@VM-8-14-centos cp_test]# touch a.txt
[root@VM-8-14-centos cp_test]# ls
a.txt
```

然后测试`-l`操作

```bash
[root@VM-8-14-centos cp_test]# cp -l a.txt a_hard
[root@VM-8-14-centos cp_test]# ll
总用量 0
-rw-r--r-- 2 root root 0 2月   5 19:58 a_hard
-rw-r--r-- 2 root root 0 2月   5 19:58 a.txt
```

通过stat查看一下命令，可以看到创建的文件属性是一样的

```bash
[root@VM-8-14-centos cp_test]# stat a.txt
  文件："a.txt"
  大小：0               块：0          IO 块：4096   普通空文件
设备：fd01h/64769d      Inode：271594      硬链接：2
权限：(0644/-rw-r--r--)  Uid：(    0/    root)   Gid：(    0/    root)
最近访问：2021-02-05 19:58:28.644866567 +0800
最近更改：2021-02-05 19:58:28.644866567 +0800
最近改动：2021-02-05 20:00:38.204437377 +0800
创建时间：-
[root@VM-8-14-centos cp_test]# stat a_hard
  文件："a_hard"
  大小：0               块：0          IO 块：4096   普通空文件
设备：fd01h/64769d      Inode：271594      硬链接：2
权限：(0644/-rw-r--r--)  Uid：(    0/    root)   Gid：(    0/    root)
最近访问：2021-02-05 19:58:28.644866567 +0800
最近更改：2021-02-05 19:58:28.644866567 +0800
最近改动：2021-02-05 20:00:38.204437377 +0800
创建时间：-
```

再测试`-s`创建软链接选项

```bash
[root@VM-8-14-centos cp_test]# cp -s a.txt a_soft
[root@VM-8-14-centos cp_test]# ll
总用量 0
-rw-r--r-- 2 root root 0 2月   5 19:58 a_hard
lrwxrwxrwx 1 root root 5 2月   5 20:05 a_soft -> a.txt
-rw-r--r-- 2 root root 0 2月   5 19:58 a.txt
```

我们在做文件备份的时候，需要将文件的属性也要备份过去，包括权限，可以使用`-p`命令

```bash
[root@VM-8-14-centos cp_test]# cp -p a.txt a.txt.back
[root@VM-8-14-centos cp_test]# ll
总用量 0
-rw-r--r-- 2 root root 0 2月   5 19:58 a_hard
lrwxrwxrwx 1 root root 5 2月   5 20:05 a_soft -> a.txt
-rw-r--r-- 2 root root 0 2月   5 19:58 a.txt
-rw-r--r-- 1 root root 0 2月   5 19:58 a.txt.back
```

我们再对软链接a_soft进行一下cp，并且通过指定`-d`和不指定`-d`对比一下

```bash
[root@VM-8-14-centos cp_test]# cp -d a_soft a_soft.back
[root@VM-8-14-centos cp_test]# ll
总用量 0
-rw-r--r-- 2 root root 0 2月   5 19:58 a_hard
lrwxrwxrwx 1 root root 5 2月   5 20:05 a_soft -> a.txt
lrwxrwxrwx 1 root root 5 2月   5 20:08 a_soft.back -> a.txt
-rw-r--r-- 2 root root 0 2月   5 19:58 a.txt
-rw-r--r-- 1 root root 0 2月   5 19:58 a.txt.back
[root@VM-8-14-centos cp_test]# cp a_soft a_soft.back1
[root@VM-8-14-centos cp_test]# ll
总用量 0
-rw-r--r-- 2 root root 0 2月   5 19:58 a_hard
lrwxrwxrwx 1 root root 5 2月   5 20:05 a_soft -> a.txt
lrwxrwxrwx 1 root root 5 2月   5 20:08 a_soft.back -> a.txt
-rw-r--r-- 1 root root 0 2月   5 20:08 a_soft.back1
-rw-r--r-- 2 root root 0 2月   5 19:58 a.txt
-rw-r--r-- 1 root root 0 2月   5 19:58 a.txt.back
```

我们看到，如果没有指定`-d`，默认只是copy了`a_soft`，并没有链接的信息，如果指定了`-d`，则是将软链接所指的源文件又一次的进行了链接。

测试一下`-u`指令，先在a.txt中增加一些内容

```bash
[root@VM-8-14-centos cp_test]# cat a.txt
I am coder
```

将`a.txt`使用`-u`指令复制到`a.txt.u`

```bash
[root@VM-8-14-centos cp_test]# cp -u a.txt a.txt.u
[root@VM-8-14-centos cp_test]# cat a.txt.u
I am coder
```

当我们修改了`a.txt`文件内容后，再次cp到`a.txt.u`时

```bash
[root@VM-8-14-centos cp_test]# cp -u a.txt a.txt.u
cp：是否覆盖"a.txt.u"？ yes
[root@VM-8-14-centos cp_test]# cat a.txt.u
I am coder
aa
```
