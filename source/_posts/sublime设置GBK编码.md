---
title: sublime设置GBK编码
date: 2021-01-09 23:24:24
categories: [tool]
cover: https://raw.githubusercontent.com/lightingsui/Pic/master/img/7d389966de9ca6163defe14a3fa2b292.jpeg
---
## sublime设置GBK编码

sublime默认是没有GBK编码的，需要手动安装

找到package control，然后选择install package

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210109231206.png)

在输入框中输入conver

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210109231347.png)

选择ConvertToUTF8，安装即可。

此时我们点击File，发现多了两个选项Set File Encoding to和Reload with Encoding。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181114141411131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDQyMzU4,size_16,color_FFFFFF,t_70)
如果你是Linux系统，此时如果我们想要把文件保存为GBK仍然做不到，会有如下提示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/201811141415554.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDQyMzU4,size_16,color_FFFFFF,t_70)
其实在插件的[文档](https://github.com/seanliang/ConvertToUTF8/blob/master/README.zh_CN.md)中已经提到，我们需要下载Codecs33。步骤同刚才下载converToUTF8。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181114142144616.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDQyMzU4,size_16,color_FFFFFF,t_70)
安装完成后重启Sublime，该功能即可使用（如果不重启，仍然会保存失败）。
