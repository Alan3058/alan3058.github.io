---
layout: post
title: [自动化发布-6.发布初探]
categories: [AutoDeploy]
tags: [AutoDeploy,发布初探]
id: [18889207447552]
fullview: false
---
前面已经将Jenkins服务搭建起来了，并且统一了项目标准规范，以下是通过一个简单标准的Gradle项目来演示Jenkins发布。
* 首先我们在Jenkins中新建一个task任务，填上task名称和描述。
![mmexport1495612296745.jpg](/assets/resources/image/20170524/1495613436552040935.jpg "1495613436552040935.jpg")


* 选择项目源代码路径，这里我们的源代码是使用gitlab管理，所以选择git选项。
![mmexport1495614580097.jpg](/assets/resources/image/20170524/1495614772234042080.jpg "1495614772234042080.jpg")

* 选择gradle版本，并设置Gradle编译任务
![mmexport1495614583076.jpg](/assets/resources/image/20170524/1495614785049009572.jpg "1495614785049009572.jpg")

* 编写本地shell和远程shell脚本。首先本地shell脚本将程序包传输到远程服务器，然后执行远程shell脚本发布启动远程应用服务器
![mmexport1495614586622.jpg](/assets/resources/image/20170524/1495614792537098458.jpg "1495614792537098458.jpg")

这样就完成了一次Jenkins应用发布配置。本次主要完成了自动化编译构建打包，并将发布程序包、启动中间服务器做成自动化，已经比之前手工发布更先进一步了。
