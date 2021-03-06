---
layout: post
title: [Tomcat源码分析—9.Ajp Connector组件（译）]
categories: [Tomcat]
tags: [tomcat,源码分析,connector]
id: [18944533799552]
fullview: false
---

# AJP Connector

Tomcat AJP Connector组件官方地址：[http://tomcat.apache.org/tomcat-8.0-doc/config/ajp.html](http://tomcat.apache.org/tomcat-8.0-doc/config/ajp.html)。

### 介绍

**AJP Connector**元素表示一个支持AJP协议的**Connector**组件。它通常被用作将Tomcat整合到现有的Apache服务器上，并且你想要Apache去处理web应用的静态内容，或者使用Apache的SSL处理。

这连接器结合
```java
jvmRoute
```属性的[Engine](http://tomcat.apache.org/tomcat-8.0-doc/config/engine.html)，可以支持负载均衡功能。

目前版本的Tomcat支持以下原生连接器：

* JK 1.2.x with any of the supported servers. See [the JK docs](http://tomcat.apache.org/connectors-doc/) for details.

* mod_proxy on Apache httpd 2.x (included by default in Apache HTTP Server 2.2), with AJP enabled: see [the httpd docs](http://httpd.apache.org/docs/2.2/mod/mod_proxy_ajp.html) for details.


**Other native connectors supporting AJP may work, but are no longer supported.**

### 特殊功能

### Proxy Support

The 
```java
proxyName
``` and 
```java
proxyPort
``` attributes can be used when Tomcat is run behind a proxy server. These attributes modify the values returned to web applications that call the 
```java
request.getServerName()
``` and 
```java
request.getServerPort()
``` methods, which are often used to construct absolute URLs for redirects. Without configuring these attributes, the values returned would reflect the server name and port on which the connection from the proxy server was received, rather than the server name and port to whom the client directed the original request.

For more information, see the [Proxy Support HOW-TO](http://tomcat.apache.org/tomcat-8.0-doc/proxy-howto.html).

### Connector Comparison

Below is a small chart that shows how the connectors differ.
Java Blocking Connector
BIOJava Nio Connector
NIOJava Nio2 Connector
NIO2APR/native Connector
APRClassname```java
AjpProtocol
``````java
AjpNioProtocol
``````java
AjpNio2Protocol
``````java
AjpAprProtocol
```Tomcat Version3.x onwards7.x onwards8.x onwards5.5.x onwardsSupport PollingNOYESYESYESPolling SizeN/A```java
maxConnections
``````java
maxConnections
``````java
maxConnections
```Read Request HeadersBlockingBlockingBlockingBlockingRead Request BodyBlockingBlockingBlockingBlockingWrite Response Headers and BodyBlockingBlockingBlockingBlockingWait for next RequestBlockingNon BlockingNon BlockingNon BlockingMax Connections```java
maxConnections
``````java
maxConnections
``````java
maxConnections
``````java
maxConnections


```


