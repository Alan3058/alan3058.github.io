---
layout: post
title: [Centos7桥接模式不能联网]
categories: [devtool]
tags: [centos,centos7,桥接模式,不能联网]
id: [13117608970682368]
fullview: false
---
### service network restart无法启动

执行service network restart出现如下错误
> Restarting network (via systemctl): Job for network.service failed. See 'systemctl status network.service' and 'journalctl -xn' for details.

排查原因：HWADDR设置错误或者未设置，这里是未设置。

通过ip addr查看mac地址，然后在/etc/sysconfig/network-scripts/ifcfg-enp0s3文件加上对应mac地址。
> HWADDR=08:00:27:CC:9F:EE

参考地址：[http://blog.csdn.net/zkja595470467/article/details/53007915](http://blog.csdn.net/zkja595470467/article/details/53007915)

### 连接外网不通

在/etc/sysconfig/network-scripts/ifcfg-enp0s3文件加上如下配置信息
```ini
BOOTPROTO=static #开启静态地址
ONBOOT=yes #开启自动启用网络连接
IPADDR=192.168.10.11 #设置静态ip
NETMASK=255.255.255.0 #子网掩码
GATWAY=192.168.32.254 #网关
DNS1=192.168.1.1  #DNS地址
```
