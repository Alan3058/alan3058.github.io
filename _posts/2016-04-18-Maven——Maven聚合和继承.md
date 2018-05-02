---
layout: post
title: [Maven——Maven聚合和继承]
categories: [开发工具]
tags: [maven,聚合,继承]
fullview: false
---
当我底下有多个Maven项目的时候，一般我们会通过对每个Maven项目进行构建，然而这种重复的点击工作将会带来很多负面情绪，令人躁动。不过Maven已经为我们预见到了该问题所在，故而提出了聚合概念，即新建一个父级pom工程，后续只需构建父级pom工程，底下所有的子Maven项目也将一一被构建。

那么何谓继承呢？写过程序的哥哥都应该知道这个东西。Maven的继承是为了解决每个子Maven项目重复引用相同的jar包依赖，即子Maven项目会继承父Maven项目的pom配置文件。

即聚合解决了重复构建的工作量，继承解决了重复引用项目jar包的代码量，并能统一子项目jar包版本的管理。

# 聚合实践

1.创建Maven项目project1。

2.创建Maven项目project2

3.创建空的Maven项目maven-parent(没有Java代码，只有一个pom.xml配置文件)。pom.xml内容如下
<groupId>com.maventest</groupId> <artifactId>maven-parent</artifactId> <version>0.0.1-SNAPSHOT</version> <packaging>pom</packaging> <!--表示该项目下有一个子Maven项目project1--> <modules> <module>../project1</module> <module>../project2</module> </modules>

4.此时对maven-parent项目进行构建，会同时构建其子模块Maven项目project1和project2。

注意:父级pom配置文件里面的packaging属性值为pom。如果子项目在父项目目录中，则module值为projectName,如果子项目和父项目在同一级，则该值为../projectName。

# 继承实践

1.新建空的父级Maven项目maven-parent(没有java代码，只有一个pom配置文件)。pom.xml内容添加如下jar包依赖和属性定义。
<properties> <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> </properties> <dependencyManagement> <dependencies> <dependency> <groupId>junit</groupId> <artifactId>junit</artifactId> <version>3.8.1</version> <scope>test</scope> </dependency> </dependencies> </dependencyManagement>

2.创建子级Maven项目project1，在其pom配置文件配置父级的坐标，并且子项目不需要groupId和version属性。如下配置

<artifactId>project1</artifactId> <parent> <groupId>com.maventest</groupId> <artifactId>maven-parent</artifactId> <version>0.0.1-SNAPSHOT</version> </parent> <dependencies> <dependency> <groupId>junit</groupId> <artifactId>junit</artifactId> </dependency> </dependencies>

如上，当子Maven项目添加新jar包时，只需指定groupId和artifactId。不需要指定版本号和依赖范围，因为子项目将从父级项目中去继承这些属性。

# 聚合和继承混合实践

通过上面两个例子总结下Maven聚合和继承的区别。

聚合是在父级pom中配置子Maven项目名称。子项目并不知道有父级Maven项目。

继承是在子Maven项目中配置父级pom坐标，并且新增的jar包依赖不需要指定版本号和依赖范围，这些属性将从父级项目中继承过来。

然而两者是可以混合起来使用的。如下案例配置

1.父级Maven项目pom配置文件内容
<groupId>com.maventest</groupId> <artifactId>maven-parent</artifactId> <version>0.0.1-SNAPSHOT</version> <packaging>pom</packaging> <name>maven-parent</name> <properties> <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> </properties> <dependencyManagement> <dependencies> <dependency> <groupId>junit</groupId> <artifactId>junit</artifactId> <version>3.8.1</version> <scope>test</scope> </dependency> </dependencies> </dependencyManagement> <modules> <module>../project1</module> </modules>

2.子级Maven项目pom配置文件内容
<artifactId>project1</artifactId> <packaging>jar</packaging> <name>project1</name> <parent> <groupId>com.maventest</groupId> <artifactId>maven-parent</artifactId> <version>0.0.1-SNAPSHOT</version> </parent> <dependencies> <dependency> <groupId>junit</groupId> <artifactId>junit</artifactId> </dependency> </dependencies>

注意：这里父级和子级的Maven项目都在同一目录下，所以module的值为../projectName。
