---
layout: post
title: [Maven——什么是Maven]
categories: [开发工具]
tags: [maven,什么是maven]
fullview: false
---
使用Maven有一段时间了，给我带来了许多帮助。比如不需要去各大网站去搜索需要的jar包，基本上在pom配置文件中添加该jar包的坐标依赖即可；打包方便，只需一个命令mvn clean package，即可完成。。。然而当有人问起我你用过maven吗？你知道maven是做什么的吗？我去，我开始疑惑了。你妹，这啥问题，maven不就是一个管理jar包的工具吗!这问题还需要回答吗？？？然而事后思考，发现这样回答很不正式，非常业余。经过翻查相关资料，找到比较合适的解释：Maven是一个管理项目依赖的构建工具。何为构建，即Java哥哥每天做的事——清理编译、编译、单元测试、打包、发布。何为项目依赖，简单点说就是管理Jar包啦。

# 项目构建？

上面一提到，项目构建是程序开发中的清理编译、编译、单元测试、打包、发布。那Maven是怎么做到的？Maven提供一系列命令来完成这些事。当我们使用如下命令来执行Maven项目，控制台将会输出调用对应插件去完成相应的事情。

mvn clean：调用clean插件去清理当前项目，即删除项目路径下的target目录

mvn compile：调用resource插件的resources命令编译项目资源文件(src/main/resources)，再调用compile插件的compile命令编译当前项目Java文件（src/main/java）。

mvn test：调用resource插件的testResources命令编译资源文件，再调用compile插件的testCompile命令编译Java文件，最后调用surefire插件的test命令执行单元测试。

mvn package：该命令相当于调用mvn compile去编译项目，再调用mvn test进行单元测试，最后调用对应打包插件（比如jar插件）将项目打包。

mvn install：该命令相当于在执行完mvn package命令后，再调用install插件将项目安装到本地Maven仓库中。

mvn deploy：该命令相当于在执行完mvn install命令后，再将项目打成的包文件上传到maven私服上，以供大家使用。

# 项目依赖?

Maven是如何管理项目的依赖呢？Maven中约定必须给每个Maven项目赋予一个坐标，并且是唯一的，否则会造成项目依赖混乱。那什么是Maven坐标呢，Maven坐标有哪些属性，是XYZ吗？大家都知道立体几何中的坐标XYZ是为了确定一个点在空间的位置，并且这个点是唯一的。其实Maven坐标作用亦是如此，为了确立当前项目在Maven仓库世界里唯一。

用过Maven的人都应该熟悉以下xml
<dependency> <groupId>org.springframework</groupId> <artifactId>spring-core</artifactId> <version>4.0.2.RELEASE</version> </dependency>

该段xml的信息就是在描述Spring的Core jar包在Maven仓库的位置。它是由Spring提供，Spring组织按照Maven的约定来定义。其中groupId我们一般称之为项目集合或项目组代号，artifactId是该项目集合下的一个项目代号，version是该项目的版本号。这三个属性是必输的，并且通过这三个属性可以确认一个项目的位置。当然Maven还定义了两个非必需属性packaging、classifier。其中packaging定义了Maven项目的打包方式，有jar、war等方式，默认为jar。classifier是属于当前项目的附属构件，比如项目文档、源码等，可忽略。
