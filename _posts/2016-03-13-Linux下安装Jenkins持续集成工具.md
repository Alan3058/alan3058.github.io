---
layout: post
title: [Linux下安装Jenkins持续集成工具]
categories: [开发工具]
tags: [linux,jenkins,安装]
fullview: false
---
对于Jenkins的定义，百度百科是这样定义的。Jenkins是一个持续集成工具，用于监控持续重复的工作。对我而言，其实就是一个专为懒人使用的自动发布工具![](http://img.baidu.com/hi/jx2/j_0007.gif)![](http://img.baidu.com/hi/jx2/j_0007.gif)![](http://img.baidu.com/hi/jx2/j_0007.gif)。像我这种不喜重复工作，浪费时间，浪费生命的人绝对得用上咯。

# 环境

操作系统：Linux centos6.5 64位

软件：Jenkins.war

Tomcat 7.0.68

Jdk 1.8

注：使用Windows访问linux时，需检查linux端口是否被防火墙拦住。

# 开始

安装Jenkins有两种方法，一种是直接输入java命令启动java -jar jenkins.war，然后访问[http://192.168.0.24:8080](http://192.168.0.24:8080)访问jenkins工程。

另外一种就是将jenkins.war放在应用服务器中间件中启动访问。下面介绍在Tomcat中间件搭建。

1.进入Jenkins官网[http://jenkins-ci.org/](http://jenkins-ci.org/)下载jenkins.war包。

2.进入apache官网[https://tomcat.apache.org/](https://tomcat.apache.org/)下载tomcat压缩包apache-tomcat-7.0.68

3.解压tomcat压缩包
$ tar -zxvf apache-tomcat-7.0.68 -C tomcat7

4.将jenkins移动到tomcat的webapps文件夹下

$ mv jenkins.war apache-tomcat-7.0.68/webapps

5.启动tomcat

$ ./apache-tomcat-7.0.68/bin/startup.sh

6.输入网址[http://192.168.0.24:8080/jenkins](http://192.168.0.24:8080/jenkins)，检查是否启动成功

如上jenkins已完成安装，具体使用可以参考相关文档或者自己点点应该就差不多会很熟悉
