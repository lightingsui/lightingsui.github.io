---
title: typora修改快捷键
date: 2021-01-12 18:40:09
categories: [tool]
cover: https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=634818545,2803921456&fm=15&gp=0.jpg
---
# typora快捷键修改

点击 File(文件) ->  偏好设置 -> 通用 -> 打开高级设置

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210112183217.png)

在弹出的文件中选择`conf.user.json`进行编辑

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210112183523.png)

向文件中`keyBinding`中添加键值对即可，格式为：

```
"在工具栏中的功能名称": "快捷键"
```

例如上面我添加的`"代码": "Alt+q"`这个。

对应的工具栏名称

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210112193705.png)

添加完成后重启typora即可生效。


