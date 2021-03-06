---
layout: post
title: [Tomcat源码分析—7.Server和Service组件（译）]
categories: [Tomcat]
tags: [tomcat,源码分析,server,service]
id: [18939539095552]
fullview: false
---

# 1.Server组件

Tomcat Server组件说明官方地址：[http://tomcat.apache.org/tomcat-8.0-doc/config/server.html](http://tomcat.apache.org/tomcat-8.0-doc/config/server.html)，如下是其大致内容。 

### 介绍

**一个Server**元素代表整个Catalina servlet容器。 因此它必须只能在
```java
conf/server.xml
```配置文件最外层，并且只有一个。它的属性代表servlet容器的整体特性。

### 属性

### 通用属性

**Server** 的所有实现都支持一下属性：
属性描述```java
className
```
Java类名，这个类必须实现
```java
org.apache.catalina.Server
```接口。如果没有指定类名，默认使用标准的实现StandardServer。
```java
address
```
TCP/IP地址，该地址是服务器等待关机命令的TCP/IP地址，如果没有指定，默认使用
```java
localhost。
```
**```java
port
```**
端口号，它是服务器等待关机命令的TCP/IP的端口号，如果设为
```java
-1
```，标识禁用关闭端口号。

Note: Disabling the shutdown port works well when Tomcat is started using [Apache Commons Daemon](http://commons.apache.org/daemon/) (running as a service on Windows or with jsvc on un\*xes). It cannot be used when running Tomcat with the standard shell scripts though, as it will prevent shutdown.bat|.sh and catalina.bat|.sh from stopping it gracefully.
**```java
shutdown
```**
该命令字符指定后，必须接收TCP/IP连接指定的端口的对应字符，才可以关闭Tomcat服务。


### 该接口提供如下方法功能去作业。


1.容器初始化、启动、停止接口。

2.获取和设置关闭容器的地址、端口和关闭容器的指令（Address、Port、Shutdown）。

3.添加删除查找Service。

![blob.png](/assets/resources/image/20170714/1500040731325018373.png "1500040731325018373.png")

### 

### 标准实现

**Server**的标准实现是**org.apache.catalina.core.StandardServer。** 它支持一下附加属性。它支持以下附件属性 (除了上述通用属性):
AttributeDescription

### 嵌套组件

一下组件可能嵌套在**Server**元素中:

* [**Service**](http://tomcat.apache.org/tomcat-8.0-doc/config/service.html) - 一个或多个service元素。

* [**GlobalNamingResources**](http://tomcat.apache.org/tomcat-8.0-doc/config/globalresources.html) - 为服务器server配置JNDI全局资源。


Server元素配置的address、port、shutdown属性，主要是为了可以关闭tomcat容器。如Address=localhost port=8005 Shutdown=SHUTDOWN。


```bash
输入 > telnet localhost 8005
   > SHUTDOWN

tomcat服务将会自动关闭
```

# 2.Service组件

### 介绍

一个**Service**元素表示一个或多个**Connector**组件共享一个[Engine](http://tomcat.apache.org/tomcat-8.0-doc/config/engine.html)组件去处理进来的请求。 一个[Server](http://tomcat.apache.org/tomcat-8.0-doc/config/server.html)元素下可以嵌套一个或多个**Service**元素。

### 属性

### 通用属性

All implementations of **Service** support the following attributes:
AttributeDescription```java
className
```
Java class name of the implementation to use. This class must implement the 
```java
org.apache.catalina.Service
```interface. If no class name is specified, the standard implementation will be used.
**```java
name
```**
**Service**的名称， 如果你使用标准的Catalina组件，它将会出现在日志信息中。 The name of each **Service** that is associated with a particular [Server](http://tomcat.apache.org/tomcat-8.0-doc/config/server.html) must be unique.


### 该接口提供如下方法功能以达到添加、删除Connector和Engine容器。

![blob.png](/assets/resources/image/20170714/1500042295852078670.png "1500042295852078670.png")

### 

### 标准实现

The standard implementation of **Service** is **org.apache.catalina.core.StandardService**. It supports the following additional attributes (in addition to the common attributes listed above):
AttributeDescription

### 嵌套组件

一个**Service**元素可能嵌套一个或多个**Connector**元素，紧接着是一个[Engine](http://tomcat.apache.org/tomcat-8.0-doc/config/engine.html)元素。


