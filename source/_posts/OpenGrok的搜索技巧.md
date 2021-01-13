---
title: OpenGrok的搜索技巧
date: 2021-01-07 17:34:59
tags: tool
categories: tool
---
# OpenGrok的搜索技巧

在OpenGrok中，有以下几种搜索栏目

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210106114452.png)

**Full Path** 为全搜索，在里面输入的字符会进行全部匹配，把文件里包含此字符的文件都列出来（包含注释等）

**Definition** 为定义搜索（主要用来搜索一些函数定义等），把包含输入的定义输出出来，例如在Definition中输入allLowerCaseAscii，结果如下

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210106114854.png)

**Symbol**为符号搜索，主要用来搜索一下成员变量等

**File Path**为路径搜索，用来搜索包含指定为字符串的路径

## 基本搜索

以上输入框中，都存在一些搜索技巧

\+ 表示包含指定字符串内容

\- 表示不包含指定字符串的内容

AND(&&) 为与的关系

OR(|| 或者 空格)为或的关系

NOT(! 或者 -)为非的关系

？代表一个字符

\* 表示代表多个字符

> 注意：AND OR NOT 必须为大写才生效。？ * 不可以作为开头字符

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210106120415.png)

+ Home 代表返回主搜索界面
+ History 目前细节未知，但猜测为与历史版本的对比
+ Annotate 目前功能未知
+ Line# 点击来显示或隐藏前面的行号
+ Navigate 展示所有的函数定义导航
+ Download 下载此页代码

## 参考链接

+ https://blog.csdn.net/yush34/article/details/102671246
+ https://www.jianshu.com/p/4b69051d4a60
