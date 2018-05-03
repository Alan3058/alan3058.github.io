---
layout: post
title: [Spring Mvc——第一个应用程序]
categories: [SpringMVC]
tags: [springmvc,源码分析,搭建]
id: [18583014866944]
fullview: false
---
# 1、介绍

Spring Mvc是一个简单灵活、高度可配置的mvc框架，是Spring为前端mvc框架的一个重要解决方案。能支持市面上大多数的视图技术，比如Jsp、Velocity和Freemarker等视图，甚至支持自定义视图实现。支持xml配置和注解两种方式，通过注解能大量简化我们的配置文件，并且与Spring IOC、 AOP无缝整合，开发者使用起来非常简单，易上手。好了不多说了，现在直接来看我们的第一个例子。

# 2、第一个应用程序

这里使用的相关开发工具是Myelipse8.5，tomcat 6，Java 6，maven 3。先使用Myeclipse创建一个Maven Web工程，这里不多说了，可以网上搜索相关资料，基本上都是傻瓜式点击。

1.修改pom.xml文件，添加spring web mvc jar包依赖

```
<!-- spring mvc包 -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
	<version>3.2.13.RELEASE</version>
</dependency>
<!-- json包 -->
<dependency>
	<groupId>org.codehaus.jackson</groupId>
	<artifactId>jackson-mapper-asl</artifactId>
	<version>1.9.4</version>
</dependency>
```

2.修改web.xml文件，在该文件中配置DispatcherServlet类和URL请求过滤，配置spring的配置文件的目录，添加spring mvc的启动监听器。DispatcherServlet类会根据我们配置的Url过滤规则来接收符合规则的请求，并将请求分发处理。ContextLoaderListener监听器实现了ServletContextListener监听器接口，它监听ServletContext实例的构建和销毁，Spring mvc把IOC容器的初始化实现都放在ContextLoaderListener监听器中。

```
<!-- 设置DispatcherServlet类和对应配置文件 -->
<servlet>
	<servlet-name>dispatcher</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring-mvc.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
<!-- 设置Url过滤 -->
<servlet-mapping>
	<servlet-name>dispatcher</servlet-name>
	<url-pattern>*.shtml</url-pattern>
</servlet-mapping>
<!-- 配置spring配置文件 -->
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath:spring.xml</param-value>
</context-param>
<!-- 配置监听器 -->
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

3.在src/main/resources目录下创建spring-mvc.xml文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
						http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
						http://www.springframework.org/schema/context 
						http://www.springframework.org/schema/context/spring-context-3.0.xsd 
						http://www.springframework.org/schema/mvc 
						http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd">

	<!-- 默认开启spring mvc注解相关配置 -->
	<mvc:annotation-driven></mvc:annotation-driven>
	<!-- spring mvc 扫描路径 -->
	<context:component-scan base-package="com.springmvctest.controller" />
	<!-- spring mvc 视图解析 -->
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="jsp/" />
		<property name="suffix" value=".jsp" />
	</bean>

</beans>
```

4.创建spring.xml文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
	http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
	http://www.springframework.org/schema/context 
	http://www.springframework.org/schema/context/spring-context-3.0.xsd
	">
</beans>
```

5.在src/main/java下创建User类。

```
package com.springmvctest.model;

public class User {

	private String name;
	private int age;
	
	public User(){
		
	}
	
	public User(String name,int age){
		this.name = name;
		this.age = age;
	}
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
}
```

6.创建UserController类

```
package com.springmvctest.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import com.springmvctest.model.User;

@Controller
public class UserController {
	
	//返回ModelAndView例子 http://localhost:8080/springmvctest/test.shtml
	@RequestMapping("test")
	public Object getTest(){
		User user = new User("zhangsan", (int) (Math.random()*50));
		ModelAndView mav = new ModelAndView();
		mav.setViewName("test");
		mav.addObject(user);
		return mav;
	}
	
}
```

7.在WebRoot文件夹下创建jsp/test.jsp页面。

```
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
	<meta http-equiv="expires" content="0">    
	<meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
	<meta http-equiv="description" content="This is my page">
	<!--
	<link rel="stylesheet" type="text/css" href="styles.css">
	-->
  </head>
  
  <body>
    test页面<br/>
    名字为：${user.name} <br/>
    年龄为${user.age}
  </body>
</html>
```

8.运行结果如下。

输入http://localhost:8080/springmvctest/test.shtml，将在test.jsp页面上展现用户信息

![1454472629700631.png](http://dl2.iteye.com/upload/attachment/0110/4584/ab1b03cc-6f49-339e-a022-72c078bbeada.png "1454472629700631.png")

# 3、简要分析 

学习过J2EE的人应该都知道Servlet。它是J2EE容器启动是最重要的一个接口类，容器的启动、处理接收的请求、容器的关闭都是在该类中定义。在该类主要一下三个方法：init(ServletConfig)容器启动时调用，service(ServletRequest,ServletResponse)容器处理请求时调用，destroy()容器关闭销毁时调用。

在J2EE框架中有一个Servlet的实现子类HttpServlet，该类主要重写了处理请求的方法service(ServletRequest,ServletResponse)，该方法将各种请求方式（get、post、put、delete、trace、options）分开处理，并将这些请求方式的处理放在对应的方法（doGet、doPost、doPut、doDelete、doTrace、doOptions）中，因此我们在基于J2EE框架开发时只需实现对应的请求方法即可。ok，暂时复习到这里了，如果需要更细的了解，可以翻查相应的资料。

在Spring mvc中，也有一个Servlet类DispatcherServlet，它继承了Servlet的子类的HttpServlet，并且重写了其容器初始化init方法、容器处理请求service方法和容器销毁destroy方法，并在其基础上扩展了一些符合Spring mvc框架的功能。

在如上配置文件spring-mvc.xml中，使用<mvc:annotation-driven>元素为IOC容器默认注册了相应类，以下是官方文档解释说明。

官方文档指出使用<mvc:annotation-driven>节点，配置使用<mvc:annotation-driven>元素时，将默认注册RequestMappingHandlerMapping、RequestMappingHandlerAdapter和ExceptionHandlerExceptionResolver类，和注册HttpMessageConverter接口的实现类。

ByteArrayHttpMessageConverter：转换数组。

StringHttpMessageConverter：转换String。

ResourceHttpMessageConverter：将所有的媒体类型转换Spring mvc的Resource实例。

SourceHttpMessageConverter：转换Source。

FormHttpMessageConverter：转换表达数据成MultiValueMap<String,String>。

Jax2RootElementHttpMessageConverter：转换Java对象成XML（需添加Jax2 jar包）。

MappingJackson2HttpMessageConverter（或MappingJacksonHttpMessageConvert）：转换成json（需添加Jackon2或Jackson jar包）。

AtomFeedHttpMessageConverter：转换成Atom（添加Rome jar包）

RssChannelHttpMessageConverter：转换成Rss（添加Rome jar包）

源码见如下附件

![](http://ctosb.com/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif)[springmvctest.zip](http://dl.iteye.com/topics/download/42903542-fcfc-3c45-8f2c-0dd28d6ccd93 "springmvctest.zip")


