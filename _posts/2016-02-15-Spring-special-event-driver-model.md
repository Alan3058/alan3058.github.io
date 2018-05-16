---
layout: post
title: [Spring特性——事件驱动模型]
categories: [Spring]
tags: [spring,特性,事件驱动]
id: [18633346514944]
fullview: false
---

# 1、Spring事件模型

事件驱动模型是一种观察者模式的典型应用，或者叫发布——订阅模型，Java中awt的事件机制和Spring的事件机制都是观察者模式的应用。

一般都是发布者有更改变动时，订阅者会接收到发布者的变动通知。

举个通用的例子网上看新闻，首先我们需要去订阅新闻，当有新的新闻时，网站会自动推送新闻给已经订阅过该新闻的用户。

新建新闻Xinwen，代码如下

```java
package com.test.springevent;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.ApplicationEvent;

public class Xinwen implements ApplicationContextAware{

	private ApplicationContext applicationContext;
	private String content;
	
	public void setApplicationContext(ApplicationContext applicationContext)
			throws BeansException {
		this.applicationContext = applicationContext;
	}

	public ApplicationContext getApplicationContext() {
		return applicationContext;
	}
	
	public void process(){
		content = "新闻：小米销售额在国内第一";
		applicationContext.publishEvent(new XinwenApplicationEvent(content));
	}
	

	
	
}
```

新建新闻事件XinwenApplicationEvent，代码如下

```java
package com.test.springevent;

import org.springframework.context.ApplicationEvent;

public class XinwenApplicationEvent extends ApplicationEvent {

	public XinwenApplicationEvent(Object source) {
		super(source);
	}

}
```

创建用户1 User1，代码如下

```java
package com.test.springevent;

import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;

public class User1 implements ApplicationListener<ApplicationEvent> {

	public void onApplicationEvent(ApplicationEvent event) {
		if(event instanceof XinwenApplicationEvent){
			System.out.println("用户1查看新闻");
			System.out.println(event.getSource());
		}
	}

}
```

创建用户2 User2，代码如下

```java
package com.test.springevent;

import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;

public class User2 implements ApplicationListener<ApplicationEvent> {

	public void onApplicationEvent(ApplicationEvent event) {
		if(event instanceof XinwenApplicationEvent){
			System.out.println("用户2查看新闻");
			System.out.println(event.getSource());
		}
	}

}
```

创建bean.xml文件，内容如下

```java
<!-- 测试Spring事件机制 -->
<bean class="com.test.springevent.User1"/>
<bean class="com.test.springevent.User2"/>
<bean id="xinwen" class="com.test.springevent.Xinwen"/>
```

创建Junit测试代码

```java
/**
 * 测试spring事件
 */
@Test
public void testSpringEvent(){
	ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml"); 
	Xinwen xinwen = (Xinwen) applicationContext.getBean("xinwen");
	xinwen.process();
}
```

测试结果如下

![](/assets/resources/image/20170705/1499240729735091870.png)

# 2、代码分析

1.在Spring的ApplicationContext篇幅中有大致提到Spring事件多路广播器ApplicationEventMulticaster的初始化和监听器ApplicationListener的注册。

2.Spring的监听器ApplicationListener实现了Java的EventListener事件监听器接口类，故而我们只需实现ApplicationListener接口。

3.Spring事件ApplicationEvent类继承了EventObject接口类，如有需要，我们只需要继承ApplicationEvent事件。

4.Spring发布事件接口类是ApplicationEventPublisher，然而ApplicationContext实现了ApplicationEventPublisher接口，故而可以直接调用ApplicationContext的PublishEvent方法发布事件。

5.跟踪ApplicationContext的发布事件方法代码可知，实际ApplicationContext容器是委托事件多路广播器ApplicationEventMulticaster来发布事件的。ApplicationEventMulticaster在IOC容器初始化时被初始化，同时会将ApplicationLisener监听器注册给事件多路广播器，当然IOC容器也支持代码式注册监听器。

6.ApplicationEventMulticaster（实现子类SimpleApplicationEventMulticaster）在发布事件时，支持异步发布，只需在ApplicationEventMulticaster实例中注入Java的Executor实例，即通过多线程来完成事件异步发布。

# 3、Spring事件异步发布

在bean.xml文件添加如下bean配置。

```java
<!-- 测试Spring事件机制 异步发布事件 -->
<bean id="taskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
	<property name="corePoolSize" value="5"></property>
</bean>
<bean id="applicationEventMulticaster" class="org.springframework.context.event.SimpleApplicationEventMulticaster">
	<property name="taskExecutor" ref="taskExecutor"></property>
</bean>
```

在User1和User2中加入打印当前线程语句。

```java
public void onApplicationEvent(ApplicationEvent event) {
	if(event instanceof XinwenApplicationEvent){
		System.out.println(Thread.currentThread());
		System.out.println("用户2查看新闻");
		System.out.println(event.getSource());
	}
}
```

测试结果如下


![](/assets/resources/image/20170705/1499240739802081406.png)

如上测试结果可知，两个用户查看新闻是不同线程，即SimpleApplicationEventMulticaster类通过多线程，来实现异步发布事件。

源码见如下附件

![](http://ctosb.com/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif)[cygoattest.zip](/assets/resources/file/20170705/1499240777940046825.zip "cygoattest.zip")


