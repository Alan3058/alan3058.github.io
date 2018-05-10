---
layout: post
title: [Java线程并发之Synchronized]
categories: [Java]
tags: [java,thread,synchronized,并发]
id: [18809511477248]
fullview: false
---

# 概念

首先我们来熟悉一下几个概念：程序、进程、线程。在这之前我其实也没有去详细了解它们，对它们仅仅只是一层朦胧美![](http://img.baidu.com/hi/jx2/j_0026.gif)![](http://img.baidu.com/hi/jx2/j_0026.gif)![](http://img.baidu.com/hi/jx2/j_0026.gif)。经过一番资料查找，才渐渐若有所思，似乎明白了什么。。。
程序进程线程程序是由一堆代码构成，是一个静态的概念，它是永久性的。进程是程序的一次运行活动，是一个动态的概念，它有自己的生命周期。线程是比进程更小的一个单元，它的出现能缓解多进程并发效率低下。线程也有自己的生命周期一个程序可以创建多个进程，一个进程可以执行多个程序，进程依赖于程序。每个进程会开辟各自独立的内存空间，这样会导致创建新进程消耗大，并且进程之间通信不方便（通信机制）。而多个线程是可以共享进程的内存空间，虽然它们各自都有自己独立的运行栈和程序计数器，但相对进程而言，创建线程消耗更小，并且同一个进程下的线程调度更加灵活，可以相互协调完成特定任务。


# 多线程并发例子

在单线程编程中，不存在所谓的资源共享，因为资源都属于这个线程的，没有谁会去抢占。然而在多线程编程的世界里，会出现各种各样的抢占资源问题，然后造成各种无法预期的后果。典型的例子就是存钱取钱的问题，比如一边循环往A账户里存钱，另一边则循环往A帐号取钱。如下

```java
package com.ctosb.sample;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Account {

	private static final int NUM = 1000;
	private String name;
	private int amount;

	public Account(String name, int amount) {
		this.name = name;
		this.amount = amount;
	}

	public void add(int money) {
		this.amount = this.amount + money;
	}

	public void del(int money) {
		this.amount = this.amount - money;
	}

	@Override
	public String toString() {
		return name + "当前金额：" + amount;
	}

	public static void main(String[] args) throws InterruptedException {
		final Account account = new Account("alan", 0);
		Runnable addRunnable = new Runnable() {
			@Override
			public void run() {
				account.add(1);
			}
		};

		Runnable delRunnable = new Runnable() {
			@Override
			public void run() {
				account.del(1);
			}
		};
		ExecutorService addExecutorService = Executors.newFixedThreadPool(50);
		ExecutorService delExecutorService = Executors.newFixedThreadPool(50);
		for (int i = 0; i < NUM; i++) {
			addExecutorService.submit(addRunnable);
			delExecutorService.submit(delRunnable);
		}
		Thread.sleep(1000);
		System.out.println(account);
		addExecutorService.shutdownNow();
		delExecutorService.shutdownNow();
	}
}
```

PS:在该例子中有两个线程，每个线程分别对alan账户进程10000次的存钱和取钱，每次操作金额1元。


最终运行多次的结果如下（如果结果不明显可以增加NUM值）

```bash
alan当前金额：-124
alan当前金额：135
alan当前金额：0
```

如上三次结果，每次结果都不一致，理论上正确金额应该为0，因为增减次数和金额相同。

分析：当A线程alan金额为0，准备增加1元，这时B线程也获取到金额0并且准备减1，最后A线程得出结果是1，而B线程得出的结果是-1，这时可能出现A结果覆盖B得到1，或者说B结果覆盖A得到最后结果-1，显然这两个结果都不是我们要的结果。。。如此恶性循环下去，导致最后的结果无法预计![](http://img.baidu.com/hi/jx2/j_0012.gif)![](http://img.baidu.com/hi/jx2/j_0012.gif)![](http://img.baidu.com/hi/jx2/j_0012.gif)

# 同步

java中提供了synchronized关键字，可以实现上例程序正常计算金额。只需在add和del方法上加上该关键字即可，如下

```java
	public synchronized void add(int money) {
		this.amount = this.amount + money;
	}

	public synchronized void del(int money) {
		this.amount = this.amount - money;
	}
```

运行结果如下


```bash
alan当前金额：0
```

ok，结果正常！！！为何加了synchronized就可以呢？

synchronized中文解释是同步，在java语义中也是同步的意思，它保证了可见性和互斥性，当线程A进入当前方法或者代码段时，会在此处入口增加一把锁，以防止其他线程访问。只有线程A退出方法释放锁，其他线程才可以进入方法执行。

**当一个线程A去访问add方法时，JVM会给该象上加上一把锁，其他线程不能访问del方法或者add方法，将会进入等待阻塞状态。只有当线程A执行完了释放了锁，其他线程才可以进入这两个方法执行（然而进入这两个方法依然只有一个线程，其余线程依旧等待。。。），这样就能保证多个线程同时更新金额导致数据错误。**


