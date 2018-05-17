---
layout: post
title: [spring springmvc mybatis maven整合二]
categories: [Java,Spring,Mybatis,SpringMVC]
tags: [spring,springmvc,mybatis,maven整合]
id: [18602787082944]
fullview: false
---

第一篇([spring springmvc mybatis maven整合一](/160204/spring-springmvc-mybatis-maven-integer1))已经将spring，mybatis，mybatis插件整合完成，接下来我们将整合spring和springmvc。

首先我们先对spring和mybatis的整合编写一个测试用例，看看是否整合成功。

# 5.junit测试

1.新增com.cygoat.service包

SysUserManager.java

```java
package com.cygoat.service;

import com.cygoat.model.SysUser;

public interface SysUserManager {

	public SysUser get(String id);

}
```

SysUserManagerImpl.java

```java
package com.cygoat.service.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.cygoat.dao.SysUserMapper;
import com.cygoat.model.SysUser;
import com.cygoat.service.SysUserManager;

@Service
public class SysUserManagerImpl implements SysUserManager {
	@Autowired
	private SysUserMapper sysUserMapper;

	public SysUser get(String id){
		return sysUserMapper.selectByPrimaryKey(id);
	}
}
```

2.junit测试代码

src/test/java下创建SysUserManagerImplTest.java文件，代码如下

```java
package com.cygoat.service;

import static org.junit.Assert.*;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.cygoat.service.SysUserManager;

public class SysUserManagerImplTest {
	
	private ApplicationContext ac;

	@Before
	public void setUp() throws Exception {
		ac = new ClassPathXmlApplicationContext("spring.xml");
	}

	@After
	public void tearDown() throws Exception {
	}

	@Test
	public void testGet() {
		SysUserManager obj = (SysUserManager)ac.getBean("sysUserManagerImpl");
		assertNotNull("not null", obj.get("1"));
	}

}
```

至此，完成了spring和mybatis的整合

# 6.spring和springMvc整合

6.1.配置spring-mvc.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
http://www.springframework.org/schema/context 
http://www.springframework.org/schema/context/spring-context-3.1.xsd 
http://www.springframework.org/schema/mvc 
http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd">

	<!-- 自动扫描controller包下的所有类，使其认为spring mvc的控制器 -->
	<context:component-scan base-package="com.alan.system.controller" />

	    <!-- 注解请求映射  -->
    <bean class="org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping">        
		<property name="interceptors">
		    <list>  
		    	<!-- <ref bean="logNDCInteceptor"/>   日志拦截器，这是你自定义的拦截器
		    	<ref bean="myRequestHelperInteceptor"/>   RequestHelper拦截器，这是你自定义的拦截器 
		    	<ref bean="myPermissionsInteceptor"/>  权限拦截器，这是你自定义的拦截器 
		    	<ref bean="myUserInfoInteceptor"/>  用户信息拦截器，这是你自定义的拦截器  -->
		    </list>        
		</property>        
	</bean>  	
	<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
		<property name="messageConverters">  
			<list>  
				<ref bean="byteArray_hmc" />  
				<ref bean="string_hmc" />  
				<ref bean="resource_hmc" />  
				<ref bean="source_hmc" />  
				<ref bean="xmlAwareForm_hmc" />  
				<ref bean="jaxb2RootElement_hmc" />  
				<ref bean="jackson_hmc" />  
			</list>  
		</property>  
	</bean>  
	<bean id="byteArray_hmc" class="org.springframework.http.converter.ByteArrayHttpMessageConverter" /><!-- 处理.. -->
	<bean id="string_hmc" class="org.springframework.http.converter.StringHttpMessageConverter" /><!-- 处理.. -->
	<bean id="resource_hmc" class="org.springframework.http.converter.ResourceHttpMessageConverter" /><!-- 处理.. -->
	<bean id="source_hmc" class="org.springframework.http.converter.xml.SourceHttpMessageConverter" /><!-- 处理.. -->
	<bean id="xmlAwareForm_hmc" class="org.springframework.http.converter.xml.XmlAwareFormHttpMessageConverter" /><!-- 处理.. -->
	<bean id="jaxb2RootElement_hmc" class="org.springframework.http.converter.xml.Jaxb2RootElementHttpMessageConverter" /><!-- 处理.. -->
	<bean id="jackson_hmc" class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter" /><!-- 处理json-->

	<!-- 避免IE执行AJAX时,返回JSON出现下载文件 -->
	<!-- <bean id="mappingJacksonHttpMessageConverter" class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"> 
		<property name="supportedMediaTypes"> <list> <value>text/html;charset=UTF-8</value> 
		</list> </property> </bean> -->

	<!-- 启动Spring MVC的注解功能，完成请求和注解POJO的映射 -->
	<!-- <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"> 
		<property name="messageConverters"> <list> <ref bean="mappingJacksonHttpMessageConverter" 
		/>json转换器 </list> </property> </bean> -->

	<!-- 对模型视图名称的解析，即在模型视图名称添加前后缀 -->
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/jsp/" />
		<property name="suffix" value=".jsp" />
	</bean>

	<!-- json转换器 -->
	<!-- <bean id="jsonConverter" class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"> 
		<property name="supportedMediaTypes" value="application/json" /> </bean> -->
</beans>
```

6.2.修改web.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	
	<!-- 编码过滤器 -->  
	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	<!-- Spring MVC servlet --> 
	<servlet>
		<description>spring mvc servlet</description>
		<servlet-name>dispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<description>spring mvc 配置文件</description>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring-mvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<url-pattern>*.shtml</url-pattern>
	</servlet-mapping>
	<session-config>
		<session-timeout>10</session-timeout>
	</session-config>
	<!-- Spring和mybatis的配置文件 -->   
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring.xml</param-value>
	</context-param>
	<!-- Spring监听器  -->  
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<!-- 防止Spring内存溢出监听器 -->  
    <listener>  
        <listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>  
    </listener> 

	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>
</web-app>
```

6.3.测试

新增SysUserController类

```java
package com.cygoat.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import com.cygoat.service.SysUserManager;

@Controller
@RequestMapping("sysUser")
public class SysUserController {

	@Autowired
	private SysUserManager sysUserManager;
	
	@RequestMapping("get")
	@ResponseBody
	public Object get(String id){
		return sysUserManager.get(id);
	}
}
```

启动tomcat，在浏览器输入[http://localhost:8080/SSM/sysUser/get.shtml?id=1](http://localhost:8080/SSM/sysUser/get.shtml?id=1)

![](/assets/resources/image/20170705/1499239829816059164.png)

至此，已完成了spring、springmvc、mybatis整合

源码见如下附件

![](/assets/resources/icon_rar.gif)[SSM.zip](/assets/resources/file/20170705/1499239937410099859.zip "SSM.zip")


