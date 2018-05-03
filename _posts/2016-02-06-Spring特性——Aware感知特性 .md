---
layout: post
title: [Spring特性——Aware感知特性 ]
categories: [Spring]
tags: [spring,特性,aware,感知]
id: [18629152210944]
fullview: false
---
Aware，即感知，是Spring IOC的一个特性，当实现了对应的Aware接口时，BeanFactory工厂会在生产bean时给bean注入对应的属性，即为该bean增强一定功能。

Aware类继承图如下

![](http://file.ctosb.com/upload/image/20170705/1499240644249036751.png)

从上图可以看出Aware的所有子类接口，以下列出几个主要子类接口作用。

ApplicationContextAware：给实现该接口的bean注入ApplicationContext容器。

ApplicationEventPublisherAware：给实现该接口的bean注入ApplicationEventPublisher对象，以供分发事件使用。

BeanClassLoaderAware：给实现该接口的bean注入该类的类加载器。

BeanFactoryAware：给实现该接口的bean注入当前BeanFactory容器，此时该bean就持有BeanFactory容器功能。

BeanNameAware：给实现该接口的bean注入bean的名称。

现在新建一个ApplicationContextUtil类，实现了BeanNameAware和ApplicationContextAware接口，代码如下
package com.test.aware; import org.springframework.beans.BeansException; import org.springframework.beans.factory.BeanNameAware; import org.springframework.context.ApplicationContext; import org.springframework.context.ApplicationContextAware; public class ApplicationUtil implements ApplicationContextAware,BeanNameAware { private ApplicationContext applicationContext; private String beanName; public void setApplicationContext(ApplicationContext applicationContext) throws BeansException { this.applicationContext = applicationContext; } public ApplicationContext getApplicationContext() { return applicationContext; } public void setBeanName(String name) { this.beanName=name; } public String getBeanName() { return beanName; } }

创建bean.xml文件，内容如下
<?xml version="1.0" encoding="UTF-8"?> <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.springframework.org/schema/beans" xmlns:aop="http://www.springframework.org/schema/aop" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd"> <!-- 测试Spring IOC Aware感知特性 --> <bean id="applicationUtil" class="com.test.aware.ApplicationUtil"/> </beans>

创建Junit代码
//*/* /* 测试Aware感知特性 /*/ @Test public void testAware(){ ApplicationContext ctx = new FileSystemXmlApplicationContext("H:\\workspaceST\\cygoattest\\src\\test\\resources\\bean.xml"); ApplicationUtil util = (ApplicationUtil) ctx.getBean("applicationUtil",ApplicationUtil.class); System.out.println(util.getApplicationContext()); System.out.println(util.getBeanName()); }

测试结果如下

![](http://file.ctosb.com/upload/image/20170705/1499240656464093836.png)

如上测试结果，最后ApplicationUtil实例被注入了ApplicationContext容器和它的名字。接下来ApplicationUtil可以操作成员变量ApplicationContext的自有的功能。

源码见如下附件

![](http://ctosb.com/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif)[cygoattest.zip](http://file.ctosb.com/upload/file/20170705/1499240694524004167.zip "cygoattest.zip")
