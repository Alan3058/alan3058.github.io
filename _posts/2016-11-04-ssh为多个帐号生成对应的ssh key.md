---
layout: post
title: [ssh为多个帐号生成对应的ssh key]
categories: [开发工具]
tags: [ssh,多个帐号,生产key]
fullview: false
---
首先有两个github帐号test1@qq.com，test2@qq.com

### 1.生成key

生成test1的key，一直下一步即可
$ ssh-keygen -t rsa -C "test1@email.com"

生成test2的key，需要修改生成文件名称如下，否则会替换前一个key

$ ssh-keygen -t rsa -C "test2@email.com"

**如下修改key的文件名称**

![blob.png]( "1478223967698006.png")

### 2.添加key到ssh agent

添加ssh key到ssh agent中
$ ssh-add ~/.ssh/id_rsa_github

可能提示错误信息：

Could not open a connection to your authentication agent.

这可以执行如下命令即可。
$ ssh-agent bash$ ssh-add ~/.ssh/id_rsa_github
