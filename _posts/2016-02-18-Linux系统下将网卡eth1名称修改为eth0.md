---
layout: post
title: [Linux系统下将网卡eth1名称修改为eth0]
categories: [开发工具]
tags: [linux,修改网卡名称,eth1,eth0]
id: [18645929426944]
fullview: false
---
正常安装Linux系统，Linux会自动识别第一张网卡为eth0，第二张网卡名称eth1。然而当我们使用虚拟机克隆技术去复制Linux系统时将有所不同，新克隆出来的系统的网卡信息名称可能是eth1。本人使用VirtualBox虚拟机克隆Centos6.5系统时出现过该问题，以下是解决方法。

1. 命令行：ifconfig -a ，将会展示eth1网卡的信息。
2. 编辑/etc/udev/rules.d/70-persistent-net.rules文件。  
找出NAME="eth0"信息（eth0的网卡）注释掉或者删掉。找出与eth1相同的mac地址的一行(NAME="eth1"),将它修改为NAME="eth0"。
3. reboot重启生效。
