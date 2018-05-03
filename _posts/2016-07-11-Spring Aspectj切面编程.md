---
layout: post
title: [Spring Aspectj切面编程]
categories: [Spring]
tags: [spring,aspectj,切面编程]
id: [18813705781248]
fullview: false
---
# 背景

最近碰到一个需求，系统运行一段时间后，由于数据量不断增长，导致系统运行某个功能时效率低下，甚至宕机。这时需要去定位导致系统缓慢的方法或者代码段。第一眼看到这个需求感觉非常简单，只需要在每个处理业务的地方记录前后时间，然后计算时间差即可。

```java
long fmtime = System.currentTimeMillis();
		insert(user);
		long totime = System.currentTimeMillis();
		System.out.println("insert耗时:" + (totime - fmtime));
```

# Spring切面编程

问题搞定，虽然加的地方有点多（大概有10多处），对于一个勤勤恳恳的码农来说问题不大。然而作为一个有理想的码农而言，重复做某件相同的事情，我感觉是非常可耻的。于是就有了通过Spring Aspectj的切面编程来完成的想法。直接见如下代码类。


```java
package com.ctosb.core.spring;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class Interceptor {

	@Pointcut("execution(* com.ctosb.core..get*(..))")
	public void pointCut() {
	}

	@Around("pointCut()")
	public void around(ProceedingJoinPoint joinPoint) throws Throwable {
		long fmtime = System.currentTimeMillis();
		joinPoint.proceed();
		long totime = System.currentTimeMillis();
		System.out.println(joinPoint + "耗时:" + (totime - fmtime));
	}
}
```

该类提供两个方法，pointCut和around方法。pointCut方法是一个空方法实现，它通过Aspectj注解定义了要拦截的方法集合，即com.ctosb.core包及子包下所有以get开头的方法（这里可以将get去掉，则代表所有方法）。around方法则表示要拦截pointCut方法（相当于拦截com.ctosb.core包及子包下所有以get开头的方法，具体好处请自行思考![](http://img.baidu.com/hi/jx2/j_0020.gif)![](http://img.baidu.com/hi/jx2/j_0020.gif)![](http://img.baidu.com/hi/jx2/j_0020.gif)），并记录该方法的耗时。

# 总结

面向切面编程通俗点讲，就是拦截某些业务处理功能，并在这些业务处理功能前后增加一些非业务性的功能逻辑，比如事务处理、日志记录、异常处理等通用性功能。这样就将业务功能和非业务功能代码进行很好的解耦，程序员只需要关注对业务的处理，不需要关注非业务功能代码处理（比如事务）。

