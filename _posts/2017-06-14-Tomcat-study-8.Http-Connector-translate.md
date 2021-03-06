---
layout: post
title: [Tomcat源码分析—8.Http Connector（译）]
categories: [Tomcat]
tags: [tomcat,源码分析,connector]
id: [18943733399552]
fullview: false
---

# Http Connector

Tomcat Http Connector组件官方地址：[http://tomcat.apache.org/tomcat-8.0-doc/config/http.html](http://tomcat.apache.org/tomcat-8.0-doc/config/http.html)。

### 介绍

**HTTP Connector**元素表示一个支持HTTP/1.1协议的**Connector**组件。It enables Catalina to function as a stand-alone web server, in addition to its ability to execute servlets and JSP pages.该组件的实例会去监听服务器上指定的TCP端口。 一个或者多个**Connectors** 可以配置在一个[Service](http://tomcat.apache.org/tomcat-8.0-doc/config/service.html)元素内，每个Connector都会转发到相关联的[Engine](http://tomcat.apache.org/tomcat-8.0-doc/config/engine.html)，让其去处理请求并创建响应。

如果你希望配置**Connector**，通过AJP协议(such as the 
```java
mod_jk 1.2.x
```connector for Apache 1.3)去连接web服务器，可参考[AJP Connector](http://tomcat.apache.org/tomcat-8.0-doc/config/ajp.html)文档。

每个传入的请求都要求一个线程去处理。如果同时收到请求比当前可用请求处理线程多，将会创建额外的线程，请求线程数不能操作配置的最大值（maxThreads属性的值）。如果依然收到更多的请求，那他们将会堆积在Connector实例中，数量不能操作配置的最大值(
```java
acceptCount
```属性值)。之后更多的同步请求都会收到“连接拒绝”错误，直到有资源可用以处理它们。

### 特殊功能

### HTTP/1.1 and HTTP/1.0 Support

此连接器支持http/1.1协议的所有必需特性,如RFC 2616所述，包括持久连接、管道、期望expectations和分块编码。 如果客户端（通常指浏览器）只支持HTTP/1.0， 连接器也能返回去支持这个协议。不需要特殊的配置也能支持这种特性。连接器也支持HTTP/1.0 keep-alive。

RFC 2616要求HTTP服务器总是让他们的响应开头使用HTTP最高版本。因此连接器总是在响应的开头返回HTTP/1.1。

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

### SSL Support

You can enable SSL support for a particular instance of this **Connector** by setting the 
```java
SSLEnabled
``` attribute to 
```java
true
```.

You will also need to set the 
```java
scheme
``` and 
```java
secure
``` attributes to the values 
```java
https
``` and 
```java
true
``` respectively, to pass correct information to the servlets.

The BIO, NIO and NIO2 connectors use the JSSE SSL whereas the APR/native connector uses OpenSSL. Therefore, in addition to using different attributes to configure SSL, the APR/native connector also requires keys and certificates to be provided in a different format.

For more information, see the [SSL Configuration HOW-TO](http://tomcat.apache.org/tomcat-8.0-doc/ssl-howto.html).

### SSL Support - BIO, NIO and NIO2

The BIO, NIO and NIO2 connectors use the following attributes to configure SSL:

### SSL Support - APR/Native

When APR/native is enabled, the HTTPS connector will use a socket poller for keep-alive, increasing scalability of the server. It also uses OpenSSL, which may be more optimized than JSSE depending on the processor being used, and can be complemented with many commercial accelerator components. Unlike the HTTP connector, the HTTPS connector cannot use sendfile to optimize static file processing.

The HTTPS APR/native connector has the same attributes than the HTTP APR/native connector, but adds OpenSSL specific ones. For the full details on using OpenSSL, please refer to OpenSSL documentations and the many books available for it (see the [Official OpenSSL website](http://www.openssl.org/)). The SSL specific attributes for the APR/native connector are:

### Connector Comparison

Below is a small chart that shows how the connectors differ.
Java Blocking Connector
BIOJava Nio Connector
NIOJava Nio2 Connector
NIO2APR/native Connector
APRClassname```java
Http11Protocol
``````java
Http11NioProtocol
``````java
Http11Nio2Protocol
``````java
Http11AprProtocol
```Tomcat Version3.x onwards6.x onwards8.x onwards5.5.x onwardsSupport PollingNOYESYESYESPolling SizeN/A```java
maxConnections
``````java
maxConnections
``````java
maxConnections
```Read Request HeadersBlockingNon BlockingNon BlockingBlockingRead Request BodyBlockingBlockingBlockingBlockingWrite Response Headers and BodyBlockingBlockingBlockingBlockingWait for next RequestBlockingNon BlockingNon BlockingNon BlockingSSL SupportJava SSLJava SSLJava SSLOpenSSLSSL HandshakeBlockingNon blockingNon blockingBlockingMax Connections```java
maxConnections
``````java
maxConnections
``````java
maxConnections
``````java
maxConnections


```


