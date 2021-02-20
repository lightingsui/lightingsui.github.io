---
title: linux指令-which-whereis
date: 2021-02-20 15:46:14
tags: search
categories: linux
---
# linux指令-which-whereis

## which

which指令用来搜索其他指令的详细位置，他搜索的路径是PATH环境变量中的的一系列路径

**命令格式：**

```bash
which [options] [--] programname [...]
```

**参数：**

| 参数 | 解释                                 |
| ---- | ------------------------------------ |
| -a   | 将PATH下的搜索到的所有结果全部列出来 |

**演示：**

搜索ls所在的位置

```bash
[sui@VM-8-14-centos ~]$ which ls
alias ls='ls --color=auto'
        /bin/ls
```

可以看到出现了一个alias，这个是别名代表输入ls则等同于输入了后面的一串

可以使用-a选项，列出ls的所有位置

```bash
[sui@VM-8-14-centos ~]$ which -a ls
alias ls='ls --color=auto'
        /bin/ls
        /usr/bin/ls
```

最后再来看一个特殊的例子

```bash
[sui@VM-8-14-centos ~]$ which history
/usr/bin/which: no history in (/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/sui/.local/bin:/home/sui/bin)
```

当搜索history命令时，找不到history命令的位置，由于history是shell的内置命令（可以使用type查看命令的类型），并不在PATH当中，因此无法搜索到

```bash
[sui@VM-8-14-centos ~]$ type history
history 是 shell 内嵌
[sui@VM-8-14-centos ~]$ type locate
locate 已被哈希 (/bin/locate)
```

## whereis

whereis会到几个特定的目录下去查找包含输入的命令位置，一般情况下，我们查找内容时都会先使用whereis或者locate，当查找不到的时候，再使用find命令去查找

**命令格式：**

```bash
whereis [options] [-BMS directory... -f] name...
```

**参数：**

| 参数 | 解释                             |
| ---- | -------------------------------- |
| -l   | 列出where去搜索的所有目录        |
| -b   | 只找二进制文件                   |
| -m   | 只找说明文件man路径下的文件      |
| -s   | 只找src下的文件                  |
| -u   | 搜索不在上述三个特定路径下的文件 |

**演示：**

查找ls所在的所有位置

```bash
[sui@VM-8-14-centos ~]$ whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

查找ls二进制可执行文件的位置以及man帮助文档的位置

```bash
[sui@VM-8-14-centos ~]$ whereis -b ls
ls: /usr/bin/ls
[sui@VM-8-14-centos ~]$ whereis -m ls
ls: /usr/share/man/man1/ls.1.gz
```

列出whereis都在哪些目录下查找内容，由于内容过多，只展示部分路径

```bash
[sui@VM-8-14-centos ~]$ whereis -l
bin: /usr/bin
bin: /usr/sbin
bin: /usr/lib
bin: /usr/lib64
bin: /etc
bin: /usr/etc
bin: /usr/games
bin: /usr/local/bin
bin: /usr/local/sbin
bin: /usr/local/etc
bin: /usr/local/lib
bin: /usr/local/games
bin: /usr/include
# 以下部分省略
```


