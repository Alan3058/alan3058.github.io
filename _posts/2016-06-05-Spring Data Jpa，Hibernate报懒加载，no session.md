---
layout: post
title: [Spring Data Jpa，Hibernate报懒加载，no session]
categories: [Spring]
tags: [spring data jpa,hibernate,懒加载,no session]
fullview: false
---
Hibernate，Java程序员都应该用过，就算没有用过估计也听过咯。本人的一个项目中使用了Spring Data Jpa加Hibernate，用过Hibernate的人应该都遇到过懒加载的异常。由于本人是JPA新手，特此记录下![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)。如下是异常信息，主要是说由于配置懒加载，导致在获取懒加载对象时，session事务已关闭。
org.hibernate.LazyInitializationException: could not initialize proxy - no Session

可通过如下两种方法来修改。

第一种方法：将懒加载模式配置成饥渴模式。

第二种方法：在web.xml文件中增加OpenSessionInView。

如果用Spring Data Jpa的话，修改如下
<filter> <filter-name>openEntityManagerInViewFilter</filter-name> <filter-class>org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter</filter-class> </filter> <filter-mapping> <filter-name>openEntityManagerInViewFilter</filter-name> <url-pattern>//*</url-pattern> </filter-mapping>

如果用Hibernate的话，修改如下
<filter> <filter-name>openSessionInView</filter-name> <filter-class>org.springframework.orm.hibernate3.support.OpenSessionInViewFilter</filter-class> </filter> <filter-mapping> <filter-name>openSessionInView</filter-name> <url-pattern>//*</url-pattern> </filter-mapping>

**总结**：如果配置成饥渴模式的话，由于数据量大，将会导致该model的相关查询异常缓慢；如果配置OpenEntityManagerInViewFilter将会扩大Session的生命周期，使得其在整个请求过程中都处于Open状态，并且影响所有的请求。在我的项目中，由于处理问题比较紧急，我使用的是OpenEntityManagerInViewFilter

**后续建议**：1.可以使用OpenSessionInViewInterceptor来代替OpenSessionInView，这样通过使用AOP将其配置到你所需要的方法层上。

2.针对当前逻辑业务，使用sql语句去查询，这样就不影响以前的代码。

**参考网址**：[http://blog.csdn.net/s_good/article/details/7411642](http://blog.csdn.net/s_good/article/details/7411642)
