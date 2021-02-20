---
title: linux指令-nl
date: 2021-02-19 11:34:29
tags: nl
categories: linux
---
# linux指令-nl

nl可以将输出的文件内容自动加上行号，并且可以将行号做比较多的显示设计，包括位数与是否自动补齐0等等功能

**命令格式：**

```bash
nl [OPTION]... [FILE]...
```

**参数：**

| 参数  | 解释                                          |
| ----- | --------------------------------------------- |
| -b a  | 对空行同样列出行号                            |
| -b t  | 对空行不列出行号                              |
| -n ln | 行号在自己字段的**左侧**显示                  |
| -n rn | 行号在自己字段的**右侧**显示，且**不自动**加0 |
| -n rz | 行号在自己字段到的**右侧**显示，且**自动**补0 |
| -w    | 指定行号占用的字节数                          |

**演示：**

`-b a`与`-b t`的区别

```bash
[root@VM-8-14-centos blog]# nl -b a /etc/issue
     1  \S
     2  Kernel \r on an \m
     3
[root@VM-8-14-centos blog]# nl -b t /etc/issue
     1  \S
     2  Kernel \r on an \m
```

`-w`默认的显示占用字节数为6，因此，当指定`-n ln`时

```bash
[root@VM-8-14-centos blog]# nl -b a -n ln /etc/issue
1       \S
2       Kernel \r on an \m
3
```

指定`-n rn`时

```bash
[root@VM-8-14-centos blog]# nl -b a -n rn /etc/issue
     1  \S
     2  Kernel \r on an \m
     3
```

可以看出区别了吧，ln在占用的6个字节的最左侧显示，rn在占用的6个字节的最右侧显示

如果想要将占用的字节空白部分补0，则可以使用rz

```bash
[root@VM-8-14-centos blog]# nl -b a -n rz /etc/issue
000001  \S
000002  Kernel \r on an \m
000003
```

修改默认的6字节

```bash
[root@VM-8-14-centos blog]# nl -b a -w 10 -n rz /etc/issue
0000000001      \S
0000000002      Kernel \r on an \m
0000000003
```

nl命令可以更好的控制输出的行号，是相对于`cat -b`与`cat -n`不足之处的补充
