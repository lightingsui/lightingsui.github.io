---
title: 设置右键-用sublime打开文件
date: 2021-01-09 23:23:59
categories: [tool]
cover: https://raw.githubusercontent.com/lightingsui/Pic/master/img/7d389966de9ca6163defe14a3fa2b292.jpeg
---
# 设置右键-用sublime打开文件
![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210112193738.png)

新建一个文本文件，命名为 sublime_addright.reg

复制以下内容到文件中：

```
Windows Registry Editor Version 5.00
[HKEY_CLASSES_ROOT\*\shell\SublimeText3]
@="用 SublimeText3 打开"
"Icon"="E:\\software\\sublime\\Sublime Text 3\\sublime_text.exe,0"

[HKEY_CLASSES_ROOT\*\shell\SublimeText3\command]
@="E:\\software\\sublime\\Sublime Text 3\\sublime_text.exe %1"


[HKEY_CLASSES_ROOT\Directory\shell\SublimeText3]
@="用 SublimeText3 打开"
"Icon"="E:\\software\\sublime\\Sublime Text 3\\sublime_text.exe,0"

[HKEY_CLASSES_ROOT\Directory\shell\SublimeText3\command]
@="E:\\software\\sublime\\Sublime Text 3\\sublime_text.exe %1"
```

注意替换路径！！！

保存双击即可发现右键菜单中多出来了 用sublime打开选项。

> 这里有一个小提示，上面的内容中如果涉及到了中文，那么一定要使用GBK编码，因为windows的注册表使用的就是中文，否则会乱码
