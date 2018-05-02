---
layout: post
title: [Linux下安装Nexus私服]
categories: [开发工具]
fullview: false
---
# 背景

玩过游戏的人都应该知道私服这玩意，同样使用过Maven的程序员哥哥妹妹都应该知道Nexus搭建私服这个事吧。maven其实是从中心服务器库去下载相关资料（如jar包，war包等），当我们通过Maven去添加一个jar包时，Maven会去中心服务器下载该jar包到本地。当一个公司里所有人都需要一个jar包时，所有人都去请求互联网，这时可能会导致公司带宽浪费。这是可以在公司设立一个私服Nexus，公司里所有人都不直接请求互联网，而是请求内网私服地址，当私服地址中没有该资料，私服地址再去请求互联网。这样可以有效降低公司网络带宽浪费，不仅如此，Nexus还是实现资料共享。

# 环境

操作系统：Linux centos6.5 64位

软件：Nexus 2.11-01

Java 1.8

注：nexus运行环境需要对应java版本的支持，本人下载的是Nexus2.11-1，至少需要jdk1.7。

# 开始

1.Java安装这里就不讲了，太简单了。

2.进入Nexus官网[http://www.sonatype.org/nexus/go](http://www.sonatype.org/nexus/go) 下载软件包nexus-2.11.01-bundle.tar.gz

3.解压nexus压缩包
$ tar -zxvf nexus-2.11.01-bundle.tar.gz -C software/nexus2

4.进入nexus下的bin目录下启动nexus。

$ ./nexus start

5.浏览器访问[http://192.168.0.24:8081/nexus](http://192.168.0.24:8081/nexus) ，将显示nexus主页信息。

6.右上角有登录按钮，默认用户名：admin 密码：admin123

如需要在Windows下访问linux，需检查8081端口是否被防火墙屏蔽掉。
