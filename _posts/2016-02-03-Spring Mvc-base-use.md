---
layout: post
title: [Spring Mvc——基本使用]
categories: [SpringMVC]
tags: [springmvc,源码分析,使用案例,基本使用]
id: [18595597778944]
fullview: false
---

注：以下例子都是建立在第一个例子的基础上修改

# 1、@ResponseBody

该注解放置在一个方法上，表示会将方法的返回值写入到Http response body内（实际处理：Spring mvc 通过HttpMessageConverter消息转换器接口类将返回值转换成对应消息，然后将消息写入到Http response body内）。

首先在第一个例子的UserController类中添加以下代码，去构建初始化数据

```java
private static Map<String,User> map = new HashMap<String,User>();
static{
	for(int i=0;i<10;i++){
		User user = new User("zhangsan"+i, (int) (Math.random()*50));
		map.put("zhangsan"+i,user);
	}
}
```

在UserController类中添加如下方法。

1.返回字符串

```java
//返回字符串处理例子 http://localhost:8080/springmvctest/helloworld.shtml
@RequestMapping("helloworld")
@ResponseBody
public Object helloworld(){
	return "helloworld";
}
```

测试结果：输入http://localhost:8080/springmvctest/helloworld.shtml，浏览器显示

helloworld

2.实体bean转json

```java
//实体bean转json http://localhost:8080/springmvctest/getUser.shtml
@RequestMapping("getUser")
@ResponseBody
public Object getUser(){
	User user = new User("zhangsan", (int) (Math.random()*50));
	return user;
}
```

测试结果：输入http://localhost:8080/springmvctest/getUser.shtml，浏览器显示

{"name":"zhangsan","age":26}

3.数组转json

```java
//数组转json http://localhost:8080/springmvctest/getUsers.shtml
@RequestMapping("getUsers")
@ResponseBody
public Object getUsers(){
	return map.values().toArray();
}
```

测试结果：输入http://localhost:8080/springmvctest/getUsers.shtml，浏览器显示

[{"name":"zhangsan1","age":0},{"name":"zhangsan2","age":1}]

4.Collection集合转json

```java
//Collection集合转json http://localhost:8080/springmvctest/getUserCollection.shtml
@RequestMapping("getUserCollection")
@ResponseBody
public Object getUserCollection(){
	return map.values();
}
```

测试结果：输入http://localhost:8080/springmvctest/getUserCollection.shtml，浏览器显示

[{"name":"zhangsan1","age":0},{"name":"zhangsan2","age":1}]

5.Map集合转json

```java
//Map集合转json http://localhost:8080/springmvctest/getUserMap.shtml
@RequestMapping("getUserMap")
@ResponseBody
public Object getUserMap(){
	return map;
}
```

测试结果：输入http://localhost:8080/springmvctest/getUserMap.shtml，浏览器显示

{"zhangsan1":{"name":"zhangsan1","age":0},"zhangsan2":{"name":"zhangsan2","age":1}} 

# 2、@PathVariable

该注解用在方法参数上，被注解的参数的值将在对应的Url路径上。

```java
//PathVariable注解例子 http://localhost:8080/springmvctest/get/zhangsan1.shtml
@RequestMapping("get/{username}")
@ResponseBody
public Object get(@PathVariable String username){
	User user = map.get(username);
	return user;
}

//PathVariable注解使用正则表达式 http://localhost:8080/springmvctest/get1/zhangsan1.shtml
@RequestMapping("get1/{username:[a-z]+[1]}")
@ResponseBody
public Object get1(@PathVariable String username){
	User user = map.get(username);
	return user;
}
```

测试结果如下

输入http://localhost:8080/springmvctest/get/zhangsan1.shtml，浏览器展示

{"name":"zhangsan1","age":0}

输入 http://localhost:8080/springmvctest/get/zhangsan1.shtml，浏览器展示

{"name":"zhangsan1","age":0}

# 3、@RequestParam

@RequestParam注解使用在方法参数上，使用它可以接收请求的参数。

```java
//RequestParam注解 http://localhost:8080/springmvctest/getByusername.shtml?username=zhangsan1
@RequestMapping("getByusername")
@ResponseBody
public Object getByusername(@RequestParam String username){
	User user = map.get(username);
	return user;
}
```

测试结果：输入http://localhost:8080/springmvctest/getByusername.shtml?username=zhangsan1，浏览器展示

{"name":"zhangsan1","age":12}

# 4、@RequestBody

@RequestBody注解使用在方法参数上，用来接收请求Body内容

```java
//RequestBody注解 http://localhost:8080/springmvctest/getRequestBody.shtml
@RequestMapping("getRequestBody")
@ResponseBody
public Object getRequestBody(@RequestBody String requestBody){
	
	return "testRequestBody:"+requestBody;
}
```

测试结果如下

url ——》 http://localhost:8080/springmvctest/getRequestBody.shtml

请求体——》test

浏览器显示结果 testRequestBody:test

测试设置如下

![1.png](/assets/resources/image/20160203/104b3e3bbe24.png "1454478098550788.png")

# 5、@ModelAttribute

@ModelAttribute可用于方法体和方法参数上，使用在方法参数上可将请求参数转换为对应model对象，使用在方法体上暂不考虑。

```java
//ModelAttribute注解 http://localhost:8080/springmvctest/getUserByUser.shtml?name=lisi&age=32
@RequestMapping(value="getUserByUser")
@ResponseBody
public Object getUserByUser(@ModelAttribute("user") User user){
	
	return user;
}
```

运行结果：输入 http://localhost:8080/springmvctest/getUserByUser.shtml?name=lisi&age=32，浏览器展示如下

{"name":"lisi","age":32}

# 6、请求和响应对象作为方法接收参数

spring mvc可让HttpServletRequest和HttpServletResponse对象作为方法的接收参数 

```java
//请求、响应作为方法参数 http://localhost:8080/springmvctest/getUserByRequest.shtml?name=wangwu&age=32
@RequestMapping(value="getUserByRequest")
@ResponseBody
public Object getUserByRequest(HttpServletRequest request , HttpServletResponse response){
	String name = request.getParameter("name");
	int age = Integer.parseInt(request.getParameter("age"));
	return new User(name , age);
}
```

运行结果：输入http://localhost:8080/springmvctest/getUserByRequest.shtml?name=wangwu&age=32，在浏览器展示如下

{"name":"wangwu","age":32}

# 7、文件上传 

spring mvc支持文件上传，并且支持多文件。spring mvc文件上传解析器是CommonsMultipartResolver（如果使用Servlet3.0，则使用StandardServletMultipartResolver解析器 ）。

1.在spring-mvc.xml配置文件下添加解析器。

```java
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>	
```

2.在pom.xml文件下添加fileupload上传jar包。

```java
<!-- 文件上传包 -->
<dependency>
	<groupId>commons-fileupload</groupId>
	<artifactId>commons-fileupload</artifactId>
	<version>1.3.1</version>
</dependency>
```

3.添加方法

```java
//上传文件 http://localhost:8080/springmvctest/upload.shtml
@RequestMapping(value="upload")
@ResponseBody
public Object upload(@RequestParam("files") MultipartFile[] files){
	StringBuffer sb = new StringBuffer();
	for(MultipartFile file : files){
		int size = (int) (file.getSize()/1024);
		sb.append("upload success,this file size is "+size+"KB.<br/>");
	}
	return sb.toString();
}
```

4.测试结果如下图

![1.png](/assets/resources/image/20160203/bb25332f07c5.png "1454478373212726.png")

# 8、@ExceptionHandler

@ExceptionHandler注解使用在方法体上，使用它将可以拦截处理该Controller类的所有的异常，其值为一个数组，即可拦截多种异常。使用注解只能拦截当前Controller类，代码不好复用，可自己实现HandlerExceptionResolver，然后在Xml配置文件中配置，达到异常统一处理

1、使用注解方法

```java
//ExceptionHandler注解例子 http://localhost:8080/springmvctest/exceptiontest.shtml
@RequestMapping("exceptiontest")
public void exceptiontest(){
	String str = null;
//		Integer.parseInt(str);//报NumberFormatException异常
	str.trim();//报NullPointerException异常
}

@ResponseBody
@ExceptionHandler(value={NullPointerException.class,NumberFormatException.class})
public Object exception(Exception e){
	return "it is error page:" + e.getMessage();
}
```

运行结果输入http://localhost:8080/springmvctest/exceptiontest.shtml，浏览器展示如下

it is error page:null

2、使用HandlerExceptionResolver接口类，自定义处理异常。

(1).创建异常处理类HandlerException，该类实现HandlerExceptionResolver接口

```java
package com.springmvctest.handler;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

public class HandlerException implements HandlerExceptionResolver {

	public ModelAndView resolveException(HttpServletRequest request,
			HttpServletResponse response, Object handler, Exception ex) {
		
		ModelAndView mav = new ModelAndView("error");
		mav.addObject("msg", ex.getMessage());
		
		return mav;
	}

}
```

(2).在spring-mvc.xml配置文件中配置HandlerException类

```java
<bean class="com.springmvctest.handler.HandlerException"></bean>
```

(3).在jsp文件夹下构建error.jsp。

```java
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
    erro页面<br/>
    异常为：${msg} <br/>

  </body>
</html>
```

(4).在UserController类中添加如下方法

```java
//异常统一处理例子 http://localhost:8080/springmvctest/exceptiontest1.shtml
@RequestMapping("exceptiontest1")
public void exceptiontest1(){
	int i = 1/0;
}
```

(5).运行结果如下

输入http://localhost:8080/springmvctest/exceptiontest1.shtml，浏览器展示如下 

erro页面

异常为：/ by zero 

源码见如下附件

![](/assets/resources/icon_rar.gif)[springmvctest.zip](http://dl.iteye.com/topics/download/fe27332b-e18c-320c-a00b-f77376ebe9db "springmvctest.zip")


