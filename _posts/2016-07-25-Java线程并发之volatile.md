---
layout: post
title: [Java线程并发之volatile]
categories: [Java]
tags: [java,thread,并发,volatile]
id: [18834677301248]
fullview: false
---
# 背景

在Java并发编程中，经常会看到volatile关键字。其实如果看过Java并发concurrent包源代码，会发现很多地方都有用到。volatile中文翻译为可变的、不稳定的，在Java编程中，它保证了变量的可见性，即一个线程对该变量做的修改，在另一个线程能读取到该变量值。

![线程内存读取.png](/ueditor/php/upload/image/20160727/1469594621923997.png "1469594621923997.png")

如图Java内存模型里对于变量x存储在主内存中，在a、b线程各自工作内存中都持有一个x变量的副本，线程每次都是对各自工作内存的x变量副本进行操作，之后虚拟机会在合适的机会将工作内存的x变量副本值回写到主内存中。

当变量x被volatile修饰时，线程内存交互如下图

![线程内存读取2.png](/ueditor/php/upload/image/20160727/1469594766150509.png "1469594766150509.png")

即线程a修改副本x的值会立马回写到主内存中，线程b读取副本x的值也都是主内存最新的。或者这样理解，a和b线程都是对主内存做操作，这样ab线程获取到x的值都是最新的(这里纯属为个人理解)。

# 操作

如下关于volatile例子

```java
package com.ctosb.sample;

public class VolatileExample extends Thread {
	// 设置类静态变量,各线程访问这同一共享变量
	private static boolean flag = true;

	// 无限循环,等待flag变为true时才跳出循环
	public void run() {
		while (flag) {
		}
	}

	public static void main(String[] args) throws Exception {
		VolatileExample t = new VolatileExample();
		t.start();
		// sleep的目的是等待线程启动完毕,也就是说进入run的无限循环体了
		Thread.sleep(100);
		flag = false;
		t.join();
	}
}
```

运行程序，程序会一直跑，并且不会停止。这个程序有两个线程，子线程t循环判断flag是否为true，为true就进行执行，而主线程main将flag标志设为false，理论上t线程会结束循环，程序停止。


**解释：t线程和main线程各自持有一个flag变量副本，main线程修改了自身flag变量副本的值，但并未将值同步给t线程的变量副本，故而t线程一直在循环。如果给flag标志增加volatile修饰，程序会正常退出。**

```java
private static volatile boolean flag = true;
```

可见性是很难测试的，只有在高频率读取变量时，才能重现。

```java
System.out.println(flag);
```

**结论**：使用volatile是可以保证变量可见性，但我们经常会误认为volatile能保证线程操作的原子性。synchronized可以保证可见性、原子性和互斥性。在一些只读场合下，或者不进行写运算，volatile的效率将会比synchronized效率更高。

扩展:原子性，happen-before原则


