---
title: shell脚本-判断变量是否为空
date: 2021-02-18 19:45:33
tags: shell
categories: shell
---
# shell脚本-判断变量是否为空

**第一种方式**

```bash
if [ -n $1 ]
then
    echo '$1 is null1'
fi
```

如果$1为**空**，则会返回`true`

**第二种方式**

```bash
if [ "" = "$1" ]
then
    echo '$1 is null2'
fi
```

**第三种方式**

```bash
if [ ! $1 ]
then
    echo '$1 is null3'
fi
```

**第四种方式**

```bash
if test -z $1
then
    echo '$1 is null4'
fi
```


