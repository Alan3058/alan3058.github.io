---
layout: post
title: [Dos命令查看端口占用及关闭进程]
categories: [devtool]
tags: [dos,查看端口占用,关闭进程]
id: [18671095250944]
fullview: false
---
1.查看8080端口被那个进程占用
> $ netstat -ano|findstr "8080"

将可能显示如下

> TCP 192.168.10.190:1983  
> 14.17.42.49:8080 ESTABLISHED 7764

2.查看进程相关信息

> $ tasklist|findstr "7764"

展示信息如下

> QQBrower.exe 7764 Console 1 120,560K

3.关闭进程

按进程号关闭进程
> $ taskkill /pid 7764 </pid 7768>

按名称关闭进程

> $ taskkill /im QQBrower.exe </im eclipse.exe>

强行关闭可添加/f；添加/t，则在关闭的时候有提示信息。
