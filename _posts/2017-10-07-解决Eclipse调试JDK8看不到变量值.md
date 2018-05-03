---
layout: post
title: [解决Eclipse调试JDK8看不到变量值]
categories: [Java]
tags: [jdk7,jdk8,eclipse调试看不到变量值,编译jdk debug jar包]
id: [38078566360940544]
fullview: false
---
在eclipse中调试JDK源代码时，变量值不会显示，这是因为JDK7和8中的rt.jar包，踢除了debug信息，以便减少rt.jar包大小。我们只需重新编译一个rt-debug.jar包即可。大致步骤如下

在eclipse中新建一个jdk项目，将源码导入进来，然后将项目导出jar包，最后将jar包拷贝到JDK安装目录下，重启eclipse即可。

以下是详细步骤

1.新建一个Java工程，命名jdk-compile。

2.右击项目—>import，选择Archive File，然后去JDK安装目录下选择源码压缩文件（我的源码包路径是D:\Devtool\Java\jdk1.8.0_144\src.zip），点击完成，此时JDK源码将会导入到工程中，并且项目会报错，忽略项目的错误即可。

3.右击项目—>export，选择JAR file，填写导出jar包名称和路径（我的导出路径是D:\jdk8-debug\rt_debug.jar），点击完成，整个项目将会导出成jar包，jar包名为rt-debug.jar。

4.将项目生成的jar包拷贝到JDK目录下，D:\Devtool\Java\jdk1.8.0_144\jre\lib\endorsed，如果endoresd文件夹不存在，则新建文件夹。

5.重启eclipse，运行调试，debug将能看到变量值。

