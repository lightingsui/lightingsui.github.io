---
title: linux指令-cut
date: 2021-02-05 11:57:38
tags: format
categories: linux
---
# linux指令-cut

linux中cut命令是常用的文本处理命令之一，通常用来截取字符，将字符切片处理，它的处理单元为一行，也就是一行一行的进行处理，最后得到我们想要的结果

**命令格式：**

```bash
cut OPTION... [FILE]...
```

**参数信息：**

| 参数         | 解释                                   |
| ------------ | -------------------------------------- |
| -d           | 指定分割符号                           |
| -f           | 选择字段                               |
| -c           | 以**字符**为单位取出固定的**字符**区间 |
| -b           | 以**字节**为单位取出固定的**字节**区间 |
| --complement | 反选我们的设定                         |

**演示：**

指定分割符号，并且执行显示的段

```bash
[root@VM-8-14-centos test]# cat /etc/passwd | cut -d":" -f1 | head -n 5
root
bin
daemon
adm
lp
```

`-f`后指定的选择为现实的段（即以分隔符分割的段），`-f1,5`为显示第一和第五段，`-f1-5`为显示第一到第五段

```bash
[root@VM-8-14-centos test]# cat /etc/passwd | cut -d":" -f1,5 | head -n 5
root:root
bin:bin
daemon:daemon
adm:adm
lp:lp
[root@VM-8-14-centos test]# cat /etc/passwd | cut -d":" -f1-5 | head -n 5
root:x:0:0:root
bin:x:1:1:bin
daemon:x:2:2:daemon
adm:x:3:4:adm
lp:x:4:7:lp
```

指定显示的字符以及字节范围

```bash
[root@VM-8-14-centos test]# cat /etc/passwd | cut  -c1-5 | head -n 5
root:
bin:x
daemo
adm:x
lp:x:
[root@VM-8-14-centos test]# cat /etc/passwd | cut  -b1-5 | head -n 5
root:
bin:x
daemo
adm:x
lp:x:
```

可以看出两者没有区别，由于英文采用的是ASCII编码，一个字节代表了一个字符，用另一文件进行举例就明白了

```bash
[root@VM-8-14-centos test]# cat cut_test
我是一个测试文件
[root@VM-8-14-centos test]# cat cut_test | cut -c1-5
我是一个测
[root@VM-8-14-centos test]# cat cut_test | cut -b1-5
我�
```

最后这个指定字节的乱码了，原因是中文采用的是`utf-8`编码，第一到第三个字节为“我”，而第三个到第六个字节才是“是”，但是我们只去了1-5，因此乱码了

```bash
[root@VM-8-14-centos test]# cat cut_test | cut -b1-6
我是
```

输出前5个字节

```bash
[root@VM-8-14-centos test]# cat /etc/passwd | cut  -b-5 | head -n 5
root:
bin:x
daemo
adm:x
lp:x:
```

输出第五个字节之后的内容

```bash
[root@VM-8-14-centos test]# cat /etc/passwd | cut  -b5- | head -n 5
:x:0:0:root:/root:/bin/bash
x:1:1:bin:/bin:/sbin/nologin
on:x:2:2:daemon:/sbin:/sbin/nologin
x:3:4:adm:/var/adm:/sbin/nologin
:4:7:lp:/var/spool/lpd:/sbin/nologin
```

关于`--complement`

```bash
[root@VM-8-14-centos test]# cat cut_test | cut -b1-6 --complement
一个测试文件
```
