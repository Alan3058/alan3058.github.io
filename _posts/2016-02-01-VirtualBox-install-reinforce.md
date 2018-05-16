---
layout: post
title: [VirtualBox安装增强功能]
categories: [开发工具]
tags: [virtualbox,安装增强功能]
id: [18574626258944]
fullview: false
---
# 环境

本地操作系统：Windows7 64位

VirtualBox版本：5.0.10 r104061

虚拟机操作系统：Centos6.5 64位

之前一直使用vmware workstation虚拟机，最近发现VirtualBox这东东，发现这玩意是开源的，而且相对来说比较小巧，100多M，出于好奇心，安装了VirtualBox，进入了漫长的探索过程。

# 执行

1. 安装VirtualBox步骤，网上一搜一大堆。

2. 由于需要虚拟机和本地机可进行拖拽操作，故而需要安装VirtualBox增强功能。

安装网上的一些安装教程，折腾了老半天，最终都没有搞定，最后点了根烟，理了理思路尴尬尴尬尴尬。。。

3. 经过查看错误日志，最后总结如下。  

首先安装VirtualBox的增强功能需要依赖make（本机已有）、gcc、kernel-headers和kernel-devel包。然而kernel-headers和kernel-devel包的版本需要和linux系统内核版本一致（输入uname -r可查看，或者rpm -qa kernel 来查看kernel版本）。

由于我这里懒得去搜寻linux对应版本的kernel-headers和kernel-devel包，我直接升级linux的kernel版本。
```bash
$ yum list  kernel*   
$ yum -y install kernel  
$ yum -y install kernel-header  
$ yum -y install kernel-devel  
$ yum -y install gcc
```

之后重启linux系统，进入linux的光盘目录（/media目录，如果没有加载需要手工加载增强功能）去启动VBOX的增强功能。

![1454469836624049.png](/assets/resources/image/20170705/1499239227956082951.png "1454469836624049.png")

最后还报了个OpenGL Support模块加载失败，由于不影响拖拽使用，故而忽略掉了，可自行处理。

![](/assets/resources/image/20170705/1499239271934061671.png)

4. VisualBox系统磁盘大小修改  

进入VisualBox安装目录
> VBoxManage.exe modifyhd e:\visualbox\Centos6.5.vdi --resize 20000

**后记：最后考虑了一下自己最原始的需求，发现其实自己的目的很简单，就是是想要虚拟机系统和本主机系统进行文件传输。。。**

**其实完全可以通过ftp文件来通讯即可，本机使用winscp。不过他没有拖拽方便****

**使用后感：小巧、系统启动快、占用资源小，符合我的口味**
