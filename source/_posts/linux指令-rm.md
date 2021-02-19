---
title: linux指令-rm
date: 2021-02-18 18:13:59
tags: rm
categories: linux
---
# linux指令-rm

rm指令通常用来删除文件或者目录，使用它删除文件或者目录，那么一切就没有了，世界就安静了，所以执行删除命令的时候一定要小心，否则会一失足成千古恨的。

**命令格式：**

```bash
 rm [OPTION]... [FILE]...
```

**参数：**

| 参数 | 解释                                 |
| ---- | ------------------------------------ |
| -f   | 强制删除，不会再出现确认删除信息     |
| -i   | 互动模式，在删除时会再次询问是否删除 |
| -r   | 递归删除，即可以删除目录             |

**演示：**

删除一个普通的文件

```bash
[root@VM-8-14-centos test]# rm aaaa
rm：是否删除普通文件 "aaaa"？y
```

当真正的执行删除的操作时，会提示是否确认删除，输入y确认即可删除。

虽然没有指定`-i`选项，但是由于`linux`别名`alias`的存在，自动加上了`-i`

```bash
[root@VM-8-14-centos test]# alias
alias rm='rm -i'
```

如果想要在执行命令的时候不使用别名，可以在命令前加上`\`，这样就会直接删除，而不会进行提示

```bash
[root@VM-8-14-centos test]# \rm bbbbbb
[root@VM-8-14-centos test]#
```

使用`rm -rf`删除目录

```bash
[root@VM-8-14-centos test]# ls
cccc  col_test.txt  cp_test  cut_test  my  my_hard  my_soft  uniq_test
[root@VM-8-14-centos test]# rm -rf cp_test/
[root@VM-8-14-centos test]# ls
cccc  col_test.txt  cut_test  my  my_hard  my_soft  uniq_test
```

指定`-r`选项会递归删除目录下的文件以及目录

另外有一个很有趣的**小例子**

```bash
[root@VM-8-14-centos test]# touch ./-aaa-
[root@VM-8-14-centos test]# ls
-aaa-  cccc  col_test.txt  cut_test  my  my_hard  my_soft  uniq_test
[root@VM-8-14-centos test]# rm -rf -aaa-
rm：无效选项 -- a
Try 'rm ./-aaa-' to remove the file "-aaa-".
Try 'rm --help' for more information.
```

创建了一个名字为`-aaa-`的文件，发现使用`rm -rf`无法删除，原因是文件的名称带有`-`，与选项向冲突，bash会混淆，因此，如果想删除以这样文件命名的文件，需要以下几种方式

```bash
# 方式一
[root@VM-8-14-centos test]# rm -rf ./-aaa-

# 方式二
[root@VM-8-14-centos test]# rm -rf -- -aaa-
```

不过，一般要尽量避免用这种方式去命名文件

**慎用rm指令！**
