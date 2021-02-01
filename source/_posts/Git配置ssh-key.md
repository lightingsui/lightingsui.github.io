---
title: Git配置ssh_key
date: 2021-01-22 11:19:30
tags: git
categories: git
---
# Git配置ssh key

ssh key的作用就是起到与github互通的作用，如果没有设置ssh key，当我们提交代码，都会要求输入账号和密码，这样很麻烦，有了ssh key，就可以在github中标识我们的git客户端，即代表了此git客户端已经获得了github的认证，再次提交时就不需要密码了。

> 此处是以github为远端仓库，例如gitlab等都是一个样，本质上github与gitlab也没有特别大的区别

## 设置

```bash
ssh-keygen -t rsa -C '邮箱'
```

设置秘钥的保存路径（默认保存在/home/username/.ssh/id_rsa下）

```bash
Enter file in which to save the key (/root/.ssh/id_rsa):
```

输入秘钥的密码（也可以不用输入），确认密码后还会让再次输入

```bash
Enter passphrase (empty for no passphrase):
```

此刻秘钥就已经生成了，需要打开刚才生成的公钥(id_rsa.pub)文件，复制里面的内容到github中

> 在这里采用的是非对称加密，而我们又在使用私钥的基础上设置了密码，设置的密码又为对称加密，可见有多谨慎

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210122101000.png)

在左边找到 SSH and GPG keys -> 点击 New SSH key

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210122101121.png)

在title中填写标题，任意。

在key中将刚才复制的内容粘贴至此即可。

> 上面生成的ssh key是依赖于ssh协议的，因此在设置远端仓库的时候应该使用ssh协议，否则我们的秘钥就不会生效，关于ssh协议，我们以后会专门总结一篇文章

![](https://raw.githubusercontent.com/lightingsui/Pic/master/img/20210122101607.png)
