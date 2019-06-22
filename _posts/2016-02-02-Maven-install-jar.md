---
layout: post
title: [Maven命令行安装Jar包到本地仓库]
categories: [devtool]
tags: [maven,本地安装jar,命令行]
id: [18566237650944]
fullview: false
---
使用Maven有一段时间了，经常会碰到需要添加的jar包无法从网络上载下来，因此需要手工将jar包下载下来，然后将jar包安装到本地仓库中，以下是安装命令的方法和解释。
```bash
$ mvn install:install-file
     -DgroupId=org.springframework.richclient   //对应maven文件的groupId
     -DartifactId=spring-richclient-vldocking    //对应maven文件的artifactId
     -Dversion=1.1.0                                      //对应maven文件的version
     -Dpackaging=jar                                     //maven文件类型
     -Dfile=spring-richclient-vldocking-1.1.0.jar  //maven文件路径，如果是当前目录，则直接写文件名
```

完整例子

> $ mvn install:install-file -DgroupId=org.springframework.richclient -DartifactId=spring-richclient-vldocking -Dversion=1.1.0 -Dpackaging=jar -Dfile=spring-richclient-vldocking-1.1.0.jar

