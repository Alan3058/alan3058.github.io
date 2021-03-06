---
layout: post
title: [dubbo服务环境搭建]
categories: [bigdata]
tags: [java,dubbo,服务,环境搭建]
id: [18662706642944]
fullview: false
---

由于项目需要，接触到dubbo这玩意，感觉其架构真心很酷。在网上搜索了下这玩意，原来是阿里的开源体系，专为解决治理企业级分布式服务，并且目前已经有很多大型互联网公司都在使用，比如当当、京东、赶集网等。So。。。听起来就感觉吊吊的样子，先来个环境的搭建耍耍吧。![](http://img.baidu.com/hi/youa/y_0005.gif)![](http://img.baidu.com/hi/youa/y_0005.gif)![](http://img.baidu.com/hi/youa/y_0005.gif)

# 环境


操作系统：Windows 7


开发软件：dubbo2.5.3，zookeeper3.4.8（dubbo的注册中心），spring 2.5.6

开发工具：eclipse mars，Java7

# 开始搭建


## 1.zookeeper注册中心搭建


1.1.下载zookeeper软件

Apache官网下载当前最新稳定版zookeeper3.4.8，下载链接[http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz](http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz) 

1.2.安装zookeeper软件

解压压缩包，进入conf文件夹中，将zoo_sample.cfg修改为zoo.cfg，具体配置元素信息详见官方文档。

1.3.启动服务

双击运行bin/zkServer.cmd文件，启动服务。或者运行一下命令启动服务

```bash
$ bin/zkServer.cmd start
```

1.4.验证是否启动成功

输入一下命令，验证是否能够连接上zookeeper注册中心

```bash
$ bin\zkCli.cmd -server localhost:2181
```

如果输出如下信息则代表启动成功


```bash
Connecting to localhost:2181
log4j:WARN No appenders could be found for logger (org.apache.zookeeper.ZooKeeper).
log4j:WARN Please initialize the log4j system properly.
Welcome to ZooKeeper!
JLine support is enabled
[zkshell: 0]
```
之后可输入help显示命令行帮助信息。

## 2.安装dubbo-admin管理中心。

2.1.下载dubbo-admin-2.5.4.war工程文件。下载链接[http://dl.download.csdn.net/down10/20141027/fb0262c5bd796677a9d69200ec1f7848.war?response-content-disposition=attachment%3Bfilename%3Ddubbo-admin-2.5.4-SNAPSHOT.war&OSSAccessKeyId=9q6nvzoJGowBj4q1&Expires=1456827655&Signature=XUdw9OohhZFOaFWQaDb0xprlrF0%3D](http://dl.download.csdn.net/down10/20140819/b767e0a89e30275c7a14b5278285f245.war?response-content-disposition=attachment%3Bfilename%3Ddubbo-admin-2.5.4.war&OSSAccessKeyId=9q6nvzoJGowBj4q1&Expires=1456827490&Signature=rcblqZ%2FJ2YTK6zdJjDdiEfiV%2FYQ%3D) 

2.2.启动dubbo管理中心

将该文件丢入tomcat目录中，启动tomcat。

2.3.验证工程是否启动成功

进入[http://localhost:8080/dubbo-admin-2.5.4](http://localhost:8080/dubbo-admin-2.5.4) 输入用户名root 密码root，则可进入。

此时dubbo-admin管理中心已搭建成功。

注意:用户名密码信息在dubbo-admin-2.5.4\WEB-INF\dubbo.properties文件中，并且需要配置zookeeper注册中心的地址，如下内容

```java
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.admin.root.password=root
dubbo.admin.guest.password=guest
```

## 3.dubbo工程构建

在这里我将工程分为三个，分别是dubbo-api、dubbo-manager、dubbo-web。其中api负责接口、manager负责接口实现、web负责前端页面和spring mvc Controller层。


3.1.首先在dubbo-api工程中新建一个接口CommonManager，代码如下。

```java
package com.ctosb.dubbo.api;

public interface CommonManager {
    public String hello(String str);
}
```

3.2.在dubbo-manager中创建CommonManager的实现类和测试启动类MainStartup。代码如下。


```java
package com.ctosb.dubbo.manager;

import com.ctosb.dubbo.api.CommonManager;

public class CommonManagerImpl implements CommonManager {

    public String hello(String str) {
        return "Hello," + str;
    }
}
```

```java
package test;

import java.io.IOException;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainStartup {

    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"applicationContext.xml"});
        context.start();
        System.in.read();
    }

}
```

3.3.dubbo-manager的pom.xml文件添加如下jar包依赖。


```java
                <dependency>
			<groupId>com.ctosb.dubbo</groupId>
			<artifactId>dubbo-api</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>dubbo</artifactId>
			<version>2.5.3</version>
		</dependency>
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.4.6</version>
		</dependency>
		<dependency>
			<groupId>com.github.sgroschupf</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.1</version>
		</dependency>
```

3.4.dubbo-manager中spring的配置文件applicationContext.xml内容如下。


```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jee="http://www.springframework.org/schema/jee"
	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
	http://www.springframework.org/schema/tx
	http://www.springframework.org/schema/tx/spring-tx-3.1.xsd
	http://www.springframework.org/schema/jee
	http://www.springframework.org/schema/jee/spring-jee-3.1.xsd
	http://code.alibabatech.com/schema/dubbo
	http://code.alibabatech.com/schema/dubbo/dubbo.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context-3.1.xsd"
	default-lazy-init="false">
	<!-- 具体的实现bean -->
	<bean id="commonManager" class="com.ctosb.dubbo.manager.CommonManagerImpl" />
	<!-- 提供方应用名称信息，这个相当于起一个名字，我们dubbo管理页面比较清晰是哪个应用暴露出来的 -->
	<dubbo:application name="dubbo_provider"></dubbo:application>
	<!-- 使用zookeeper注册中心暴露服务地址 -->
	<dubbo:registry address="zookeeper://127.0.0.1:2181"
		check="false" subscribe="false" register=""></dubbo:registry>
	<!-- 要暴露的服务接口 -->
	<dubbo:service interface="com.ctosb.dubbo.api.CommonManager"
		ref="commonManager" />
</beans>
```

3.5.在dubbo-web工程中也新建一个测试启动类MainStartup,代码如下

```java
package test;
import java.io.IOException;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.ctosb.dubbo.api.CommonManager;

public class MainStartup {

    public static void main(String[] args) throws IOException {
        ApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"applicationContext.xml"});  
        CommonManager commonManager = (CommonManager) context.getBean("commonManager"); //
        System.out.println(commonManager.hello("world"));
        System.in.read();
    }

}
```

3.6.dubbo-web工程的pom.xml文件添加如下依赖jar包


```java
                <dependency>
			<groupId>com.ctosb.dubbo</groupId>
			<artifactId>dubbo-api</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>dubbo</artifactId>
			<version>2.5.3</version>
		</dependency>
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.4.8</version>
		</dependency>
		<dependency>
			<groupId>com.github.sgroschupf</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.1</version>
		</dependency>
```

3.7.添加dubbo-web工程的spring配置文件applicationContext.xml，内容如下

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans  
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://code.alibabatech.com/schema/dubbo  
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd  
        ">

	<!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
	<dubbo:application name="hehe_consumer" />

	<!-- 使用zookeeper注册中心暴露服务地址 -->
	<!-- <dubbo:registry address="multicast://224.5.6.7:1234" /> -->
	<dubbo:registry address="zookeeper://127.0.0.1:2181" />

	<!-- 生成远程服务代理，可以像使用本地bean一样使用demoService -->
	<dubbo:reference id="commonManager"
		interface="com.ctosb.dubbo.api.CommonManager" />

</beans>
```

3.8.启动dubbo-manager工程，将在dubbo管理中心看到已经注册了一个服务。

3.9.启动dubbo-web工程，输出启动成功日志。

如此dubbo服务基本环境已搭建完成。当然在dubbo-manager和dubbo-web中都没有用spring注解方式，dubbo-web中也没有用到spring mvc相关类，读者可自行扩展。

总结一下其实大概就以下几个步骤。

1.搭建zookeeper注册中心

2.搭建dubbo-admin管理中心，这里需要配置zookeeper注册中心的地址。

3.构建dubbo-api接口工程，可让其他工程去调用依赖。

4.构建dubbo-manager接口实现工程，该工程的将注册到zookeeper注册中心。

5.构建dubbo-web工程，该工程通过远程调用dubbo-api接口实现类的方法。


