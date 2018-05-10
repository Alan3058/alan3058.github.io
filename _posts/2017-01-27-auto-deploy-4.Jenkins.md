---
layout: post
title: [自动化发布-4.Jenkins环境]
categories: [自动化发布]
tags: [自动化发布,jenkins,搭建]
id: [18880818839552]
fullview: false
---
Jenkins是一个开源的Java项目，他使软件持续集成变成更加简单，对于运维部署人员来说是个相当不错的免费软件。他的扩展性也是非常好的，在他的插件仓库中有大量的实用插件。比如代码检查、代码覆盖率、ssh、邮件插件等，都是非常酷炫实用的。

# 安装Jenkins

第一种方式：到Jenkins官网地址[https://jenkins.io/index.html](https://jenkins.io/index.html)下载Jenkins，由于Jenkins本身也是一个Java Web项目，所以下载下来的是一个标准的war包，可以直接在tomcat等中间件中运行，也可以直接运行（内置jetty）。

在这里我是使用自带的tomcat容器，然后直接运行tomcat容器，之后可以通过web浏览器访问Jenkins。

第二种方式：Docker方式安装Jenkins
> $ docker run --name jenkins -p 8080:8080 -v /app/jenkins_data:/var/jenkins_home jenkins

可能会报权限问题。

参考地址[http://blog.csdn.net/minicto/article/details/73539986](http://blog.csdn.net/minicto/article/details/73539986)，执行如下命令（通过root用户启动），可完成安装。
> $ docker run --name jenkins -p 8080:8080 -u 0 -v /app/jenkins_data:/var/jenkins_home jenkins

# 安装Jenkins插件

接上着上节的发布案例来讲，现在我们要从拉取代码，到发布应用这段都通过Jenkins完成。首先需要从远程代码仓库拉取代码，打包代码，然后执行推送代码到远程应用中，重启远程应用。这里我们至少需要Gitlab plugin、本地shell、ssh plugin。

Jenkins插件地址：[https://plugins.jenkins.io/](https://plugins.jenkins.io/)。

Jenkins web端提供了直接搜索、安装、卸载和更新Jenkins插件的功能。以下是我安装的主要几个Jenkins插件。  

|插件名称|作用|
|-|-|
| [Git plugin](http://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin) | |
| [Git client plugin](http://wiki.jenkins-ci.org/display/JENKINS/Git+Client+Plugin) | Git插件 |
| [SSH plugin](http://wiki.jenkins-ci.org/display/JENKINS/SSH+plugin) | ssh插件 |
| [Checkstyle Plug-in](http://wiki.jenkins-ci.org/x/GYCGAQ) | 检查代码规范 | 
| [FindBugs Plug-in](http://wiki.jenkins-ci.org/x/GYAs) | 检查代码bug |
| [JaCoCo plugin](https://wiki.jenkins-ci.org/display/JENKINS/JaCoCo+Plugin) | 检查代码覆盖率| 
| [Static Analysis Collector Plug-in](http://wiki.jenkins-ci.org/x/tgeIAg) | 收集检测结果（包括Checkstyle、Findbugs、Jacoco），统计生成报表|
| [PMD Plug-in](http://wiki.jenkins-ci.org/x/GAAHAQ) | |
| [Email Extension Plugin](http://wiki.jenkins-ci.org/display/JENKINS/Email-ext+plugin) | |
| [Email Extension Template Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Email-ext+Template+Plugin) | 邮箱扩展插件，可定制模版 |
| [Gitlab Hook Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Gitlab+Hook+Plugin)| |
| [Gradle Plugin](http://wiki.jenkins-ci.org/display/JENKINS/Gradle+Plugin)| |

**注：如果公司内部服务器不能访问外网，我们可以下载Jenkins插件离线包（注意插件之间的依赖）；然后将离线包置入Jenkins的工作目录下（默认为~/.jenkins/.plugins）；最后重启Jenkins即可。我的安装方式其实是本地搭建一个Jenkins，然后本地下载安装完Jenkins后，将本地的Jenkins插件包传到公司服务器的Jenkins工作目录下。**
