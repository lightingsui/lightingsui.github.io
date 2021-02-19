---
title: Vim设置tab缩进的数量
date: 2021-02-03 15:56:21
tags: Vim
categories: tool
---
# Vim设置tab缩进的数量

vim编辑器默认tab缩进8个空格，需要我们手动修改为缩进4个空格

在redhat下，vim配置文件在`/etc/vimrc`，ubuntu下，配置文件在/etc/vim/vimrc，

找到配置文件，在配置文件的末尾加上`set tabstop=4`

重新启动vim，再次使用tab即缩进4个空格
