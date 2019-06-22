---
layout: post
title: [模板模式]
categories: [DesignPattern]
tags: [DesignPattern,模版模式,模版方法]
id: [1576258609610752]
fullview: false
---

模板模式是指将某种复杂的处理程序抽象成多道工序，并且实现顺序固定。然后让子类去实现对应的未实现工序。该模式被应用在很多开源的框架里，比如Tomcat的生命周期Lifecycle，Spring的数据库操作事务。

模板模式包含两个角色:抽象主体(Subject)，实际主体(ConcreteSubject)。它们的类图如下

![blob.png](/assets/resources/image/20170708/1499528725358003375.png "1499528725358003375.png")

编写一个实例，模拟数据库事务处理。

```java
public abstract class Subject {
	public void process() {
		System.out.println("新建一个数据库连接");
		System.out.println("开启事务，并创建一个事务");
		execute();
		System.out.println("提交事务");
		System.out.println("关闭该数据库连接");
	}

	protected abstract void execute();

	public static void main(String[] args) {
		Subject subject = new ConcreteSubject();
		subject.process();
	}
}

class ConcreteSubject extends Subject {
	protected void execute() {
		System.out.println("insert into 执行插入语句");
	}
}
```


