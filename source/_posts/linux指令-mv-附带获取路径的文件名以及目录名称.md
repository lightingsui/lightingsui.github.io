---
title: linux指令-mv-附带获取路径的文件名以及目录名称
date: 2021-02-18 20:39:24
tags: mv
categories: linux
---
# linux指令-mv-附带获取路径的文件名以及目录名称

mv指令的作用在于移动文件或目录，以及改名操作

**命令格式：**

```bash
mv [OPTION]... [-T] SOURCE DEST
mv [OPTION]... SOURCE... DIRECTORY
mv [OPTION]... -t DIRECTORY SOURCE...
```

**参数：**

| 参数 | 解释                                         |
| ---- | -------------------------------------------- |
| -f   | 强制执行，目标文件已经存在，也不会进行提示   |
| -i   | 交互式进行，目标文件已经存在则会提示是否覆盖 |
| -u   | 只有源文件比目标文件新时才会更新             |

**演示：**

将一个文件启动到其他文件夹中

```bash
[root@VM-8-14-centos test]# mkdir dir
[root@VM-8-14-centos test]# ls
auto_hexo.sh  btest  cccc  col_test.txt  dir  my  my_hard  my_soft  select_avai_null.sh  uniq_test
[root@VM-8-14-centos test]# mv btest dir
[root@VM-8-14-centos test]# cd dir/
[root@VM-8-14-centos dir]# ls
btest
```

对文件改名

```bash
[root@VM-8-14-centos dir]# ls
btest
[root@VM-8-14-centos dir]# mv btest ctest
[root@VM-8-14-centos dir]# ls
ctest
[root@VM-8-14-centos dir]#
```

mv默认以-i交互式运行，这是由于别名机制

```bash
[root@VM-8-14-centos dir]# alias
alias mv='mv -i'
```

## 获取路径的文件名以及目录名称

```bash
[root@VM-8-14-centos dir]# basename ctest
ctest
[root@VM-8-14-centos dir]# dirname /tmp/test/dir/ctest
/tmp/test/dir
```


