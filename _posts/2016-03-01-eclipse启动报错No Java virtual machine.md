---
layout: post
title: [eclipse启动报错No Java virtual machine]
categories: [开发工具]
tags: [eclipse,启动报错,No Java virtual machine]
fullview: false
---
eclipse启动报错，A Java Runtime Environment(JRE) or Java Development Kit(JDK) must be available in order to run Eclipse.No Java virtual machine was found after searching the following location:D:\devTools\eclipse\jre\bin\javaw.exe javaw.exe in your current PATH.大致意思是报找不到Java jdk或jre。如下图

![9(X2JEPGQ{_9M6DB2EPTWE5.png]( "1456803730613829.png")

修改eclipse.ini配置，在-vmargs上面添加-vm Java路径/bin/javaw.ex，如下。
-vm D:/devTools/Java/jdk1.7.0_80/bin/javaw.exe -vmargs -Dosgi.requiredJavaVersion=1.6 -Xms40m -Xmx512m
