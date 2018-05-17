---
layout: post
title: [自动化发布-3.Nexus仓库搭建]
categories: [自动化发布]
tags: [自动化发布,nexus仓库]
id: [18876624535552]
fullview: false
---
Nexus是一个maven私服仓库，他的安装方式也有两种，一种是离线包安装，另外一种是docker方式安装，这里我使用的是docker方式安装。

### 离线包安装

1.下载nexus安装包
> $ wget http://www.sonatype.org/downloads/nexus-latest-bundle.tar.gz

2.解压安装包

> $ tar -zxvf nexus-latest-bundle.tar.gz

3.进入nexus的bin目录，启动nexus

> $ ./nexus start

4.停止nexus

> $ ./nexus stop

### Docker方式安装

首先我们需要安装docker引擎，安装docker步骤详见[自动化发布-docker安装](/170212/auto-deploy-8.Docker-install)。

之后直接安装Nexus镜像
> $ docker run -d -p 8081:8081 --name nexus sonatype/nexus:oss

参考:[自动化发布-docker安装](/170212/auto-deploy-8.Docker-install)
