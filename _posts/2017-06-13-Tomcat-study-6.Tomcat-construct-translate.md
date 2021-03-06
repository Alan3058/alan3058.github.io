---
layout: post
title: [Tomcat源码分析—6.Tomcat体系架构官方概述（译）]
categories: [Tomcat]
tags: [tomcat,源码分析,官方文档,体系架构]
id: [18935344791552]
fullview: false
---

如下图，表示Tomcat重要组件图之间的关联关系。Server代表Tomcat的整个容器，并且它只能存在一个实例，它内部可以包含一个或多个Service，Service中包含两个主要的组件Connector连接器和Container容器。连接器Connector负责接收请求连接，并创建request和response实例，将request和response传递给Container容器，交由容器处理请求并响应客户端。


![mmexport1498065837332.jpg](/assets/resources/image/20170621/1498065853759056339.jpg "1498065853759056339.jpg")

### Server

在Tomcat设计中，一个[Server](http://tomcat.apache.org/tomcat-8.0-doc/config/server.html)代表整个容器。Tomcat提供[Server interface](http://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/Server.html)接口一个默认实现，一般很少会由用户自定义。

### Service

[Service](http://tomcat.apache.org/tomcat-8.0-doc/config/service.html)是一个中介容器，他生存在Server里面，并且将一个或多个Connectors连接到一个Engine上。Service元素很少由用户自定义实现，因为默认实现简单和功能足够。[Service interface](http://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/Service.html).

### Engine

[Engine](http://tomcat.apache.org/tomcat-8.0-doc/config/engine.html)代表一个特定Service的请求处理通道。一个Service可能有多个Connectors，Engine可以接收和处理来自于connectors的所有请求，将响应返回到适当的连接器，以便传输到客户端。[Engine interface](http://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/Engine.html)可能会被自定义实现，但该做法也不常用。

注： the Engine may be used for Tomcat server clustering via the jvmRoute parameter. Read the Clustering documentation for more information.

### Host

[Host](http://tomcat.apache.org/tomcat-8.0-doc/config/host.html)关联了一个网络名称，比如www.yourcompany.com。一个Engine可能包含多个hosts，并且Host元素也支持网络别名, 比如yourcompany.com and abc.yourcompany.com。用户很少去自定义[Hosts](http://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/Host.html)，因为[StandardHost implementation](http://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/core/StandardHost.html)默认实现提供了比较完善的功能。

### Connector

Connector处理与客户端的通信。Tomcat中可以有多个可用的connectors。包含[HTTP connector](http://tomcat.apache.org/tomcat-8.0-doc/config/http.html)（它用于大多数HTTP传输，特别是在将Tomcat作为独立服务器运行时），和[AJP connector](http://tomcat.apache.org/tomcat-8.0-doc/config/ajp.html)（它实现了AJP协议，被用作去连接其他的web服务器，比如Apache HTTPD服务器)。 Creating a customized connector is a significant effort.

### Context

[Context](http://tomcat.apache.org/tomcat-8.0-doc/config/context.html)代表一个web应用。一个Host可以包含多个contexts，每个context都对应唯一的路径。 [Context interface](http://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/Context.html) may be implemented to create custom Contexts, but this is rarely the case because the [StandardContext](http://tomcat.apache.org/tomcat-8.0-doc/api/org/apache/catalina/core/StandardContext.html) provides significant additional functionality.

参考Tomcat官方架构描述原文地址

[http://tomcat.apache.org/tomcat-8.0-doc/architecture/overview.html](http://tomcat.apache.org/tomcat-8.0-doc/architecture/overview.html) 


