---
layout: post
title: [Java线程方法destroy、stop、suspend、resume]
categories: [Java]
tags: [java,destroy,stop,suspend,resume]
id: [18843065909248]
fullview: false
---
**destroy方法**

该方法不支持,会抛出NoSuchMethodError异常。

**stop方法**

stop方法执行后，会立即释放当前锁，并且立即停止线程执行，可能完成数据不一致，或者数据损坏。

**suspend和resume方法**

这两个方法是成对出现。suspend并不会释放锁，必须调用该线程的resume方法才能唤醒线程，并且容易发生死锁。

* 案例分析


```java
package com.ctosb.sample;

public class ThreadMethod {

	public static void main(String[] args) throws InterruptedException {
//		stop();
		suspendAndResume();
	}

	public static void stop() {
		Thread thread = new Thread() {
			@Override
			public void run() {
				for(int i=0;i<1000;i++){
					System.out.println(i);
					try {
						Thread.sleep(20);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}

		};
		thread.start();
		try {
			Thread.sleep(50);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		thread.stop();
	}
	
	public static void suspendAndResume() {
		Thread thread = new Thread("suspendAndResume") {
			@Override
			public void run() {
				for(int i=0;i<1000;i++) {
					System.out.println(i);//1
					try {
						Thread.sleep(20);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			
		};
		thread.start();
		try {
			Thread.sleep(40);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		thread.suspend();
		System.out.println(1);//2
		thread.resume();
	}

}
```

**stop方法分析：**该例子的stop方法，执行后将会在不定次数的循环后停止，for循环并不会完整结束。如果for循环里是在更新一个耗时的对象，那么在执行stop方法后，将会立即停止for循环处理，可能会导致数据损坏。

**suspendAndResume方法分析:**程序如果正常执行的话是如下，suspend方法调用后只会暂停suspendAndResume线程，但并不会释放锁，然而之后主线程在打印一段语句后，就去唤醒suspendAndResume线程，之后程序应该正常结束。然而实际运行结果可能会发生死锁，如下通过jstack工具查看线程运行情况结果如下:suspendAndResume线程一直处于runable状态，main线程处于block状态。

在System.out.println方法中有sychronized同步语句块，也就是说打印语句需要同步加锁。设想如下场景：**suspendAndResume线程在进入代码标记行//1，main线程调用suspendAndResume线程的暂停方法，这时suspendAndResume线程虽然暂停但还是持有打印方法的锁，之后main线程进入代码标记行//2由于得不到打印方法的锁而阻塞，这样之后的唤醒suspendAndResume线程语句将不能执行，导致死锁。**

