---
layout: post
title: [赶集网Java面试题]
categories: [Java,面试]
tags: [赶集网,java 面试题,面试]
id: [18805317173248]
fullview: false
---

这两天一哥们去了赶集网面试，遇到一牛逼面试题，当时他是答错了，然后拿来给我，让我帮他一起研究。我拿来一看感觉很简单，随手就写下了答案。哥们笑道原来你也错了，哈哈。。。原题如下

```java
package com.ctosb.sample;

public class Parent {

	private String name = "parent";
	public static String string = "parentStatic";
	static {
		System.out.println(string);
	}

	public Parent() {
		print();
	}

	public void print() {
		System.out.println(name);
	}

	static class Sub extends Parent {
		private String name = "sub";
		public static String string = "subStatic";
		static {
			System.out.println(string);
		}

		public Sub() {
			print();
		}

		public void print() {
			System.out.println(name);
		}
	}

	public static void main(String[] args) {
		Parent parent = new Sub();
	}

}
```

正确运行结果是

```bash
parentStatic
subStatic
null
sub
```

呵呵。。。第一眼看到这结果，彻底晕菜了![](http://img.baidu.com/hi/jx2/j_0025.gif)![](http://img.baidu.com/hi/jx2/j_0025.gif)![](http://img.baidu.com/hi/jx2/j_0025.gif)，首先前两个输出应该是ok，唯一难以明白的就是第三个输出，居然是null。

经过一番琢磨，解释该问题的原因，应该就是新建一个对象，调用的顺序问题。如下是new一个对象，类的调用顺序。理解了这些，基本上答案已经出来了一半了。

父类静态代码块->子类静态代码块->父类属性初始化->父类构造函数->子类属性初始化->子类构造函数。

造成null的结果是在父类构造函数阶段产生的，即父类Parent的构造函数在调用print方法时，由于print方法被子类Sub重写，故而调用子类的print方法，在该方法中会打印name属性，由于子类的name属性这是还没有初始化，故而打印出来的是null。


