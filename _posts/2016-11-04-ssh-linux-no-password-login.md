---
layout: post
title: [ssh实现两台linux免密码登录]
categories: [devtool,Linux]
tags: [ssh,免密登录]
id: [18851458711552]
fullview: false
---
# 背景

在多台linux中，经常会有scp的拷贝操作，但是需要登录授权。以前一直都是通过用户名密码登录，但是一般生产环境的密码极其复杂，每次都输入的话会崩溃的。通过度娘，找到了ssh免密码登录。

# 操作

现有两台Linux机器C和S，需要实现C登录S机器免密码。这里C指客户端，S指服务端。

1. 客户端生成ssh key  
输入如下命令，一直回车，即可生成ssh key
> $ ssh-keygen -t rsa

2. 拷贝客户端的公钥到服务器上
> $ scp id_rsa.pub test@10.11.11.9:~/.ssh/authorized_keys

**注意：这样设置完后可能还是会出现需要输入密码，那么可以检查下服务端的.ssh文件夹和authorized_keys文件的权限设置。**

**.ssh文件夹应该为700，authorized_keys文件为600，测试644也可以。**
