---
layout: post
title: [单例模式]
categories: [Java,DesignPattern]
tags: [DesignPattern,单例模式,并发]
id: [18838871605248]
fullview: false
---

单例模式在日常开发中使用频率算是非常高的，而且也是是面试官最爱问的，单例模式属于创建型模式，是所有设计模式中最简单的一种，也可以说是最麻烦的一种。它的目的是让JVM只生产一个对象，如果要考虑并发的情况，那么就可能需要额外的代码去处理并发。

* **饥饿式模式的单例写法**


```java
package com.ctosb.sample;

public class Singleton {

	private static Singleton instance = new Singleton();

	private Singleton() {
	}

	public static Singleton getInstance() {
		return instance;
	}
}
```

这种写法有一种弊端，就是不管你需不需要这个对象，程序启动时虚拟机都会帮您创建该对象，可能增加内存的消耗，故而本人一般不采用。


* **懒汉式模式的写法**


```java
package com.ctosb.sample;

public class Singleton {

	private static Singleton instance;

	private Singleton() {
	}

	public static Singleton getInstance() {
		if (instance == null) {
			instance = new Singleton();
		}
		return instance;
	}
}
```

这种写法在单线程处理的情况下正常，但在多线程并发的情况下就可能会创建多次对象。可对方法加锁达到同步(即在方法上增加sychronized关键字修饰)，达到只允许一个线程进入方法创建对象。然而这样会导致并发获取对象时性能低下，这样就诞生了多重判断单例模式写法。

* **多重判断单例模式**


```java
package com.ctosb.sample;

public class Singleton {

	private static Singleton instance;

	private Singleton() {
	}

	public static Singleton getInstance() {
		if (instance == null) {//1
			synchronized (Singleton.class) {
				if (instance == null) {
					instance = new Singleton();//2
				}
			}
		}
		return instance;
	}
}
```

这样看来，通过双重判断法即解决了效率低下的问题。然而可能会出现如下问题，当线程a执行在//2代码时，首先会给instance实例分配内存，然后初始化这个变量，如果这时线程a释放锁，线程b执行到//1代码时，会直接返回instance实例，这时返回的实例其实只分配了内存，还未初始化完的对象，可能导致系统崩溃等不可预知的错误。

那还有啥方法吗？答案是有，使用内部类单例方式。

* **内部类单例(最终解决方案)**


```java
package com.ctosb.sample;

public class Singleton {

	private Singleton() {
	}

	private static class Inner {
		private static Singleton instance = new Singleton();
	}

	public static Singleton getInstance() {
		return Inner.instance;
	}
}
```

如上instance实例只有第一次调用方法时初始化，也就是说由JVM来保证实例只被初始化一次。

参考文章[http://jiangzhengjun.iteye.com/blog/652440](http://jiangzhengjun.iteye.com/blog/652440) 


