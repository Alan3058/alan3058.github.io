---
layout: post
title: [自动化发布-8.Docker安装]
categories: [AutoDeploy]
tags: [AutoDeploy,docker,docker安装]
id: [18897596055552]
fullview: false
---
随着这两年容器的兴起，docker容器化越来越受技术人员追捧，仅仅3年的时间内docker的迅速崛起。按照docker官方文档，安装docker主要有三种方式：在线安装、离线rpm包安装、离线tgz压缩包安装。

## 在线安装

1. 安装yum-utils
>$ yum install -y yum-utils

2. 添加docker.repo
>$ yum-config-manager \ --add-repo \ https://docs.docker.com/engine/installation/linux/repo_files/centos/docker.repo

3. 更新yum包索引
>$ yum makecache fast

4. 安装docker引擎
>$ yum -y install docker-engine

5. 查看docker引擎版本。你也可以去查看docker引擎版本，以便去安装指定版本的docker引擎
>$ yum list docker-engine.x86_64 --showduplicates |sort -r

6. 安装指定docker引擎版本
>$ yum -y install docker-engine-<VERSION_STRING>

7. 启动docker
>$ systemctl start docker

8. 验证docker

>$ docker run hello-world

## 离线包rpm安装

1. 进入[https://yum.dockerproject.org/repo/main/centos/](https://yum.dockerproject.org/repo/main/centos/)网址下载docker引擎。

2. 安装docker引擎
>$ yum -y install <docker-package-name>.rpm

3. 启动docker
>$ systemctl start docker

4. 验证docker
>$ docker run hello-world

## 离线tgz包安装

1. 查找并下载docker tgz包路径，详见[文档](https://github.com/docker/docker/releases)

2. 解压压缩包
>$ tar -xzvf <FILE>.tar.gz

3. 移动docker文件到usr/bin中，方便直接执行docker命令。也可以将docker文件夹加到环境变量中。
>$ cp docker//* /usr/bin/

4. 启动docker
>$ dockerd &

5. 验证docker
>$ docker run hello-world

参考文档：[docker官方安装文档](https://docs.docker.com/engine/installation/linux/centos/)

[docker官方离线tgz包安装文档](https://docs.docker.com/engine/installation/binaries/)
