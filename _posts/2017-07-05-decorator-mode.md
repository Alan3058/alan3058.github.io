---
layout: post
title: [装饰者模式]
categories: [DesignPattern]
tags: [DesignPattern,装饰者模式]
id: [1629655404642304]
fullview: false
---

装饰者模式是指在不改变目标类源代码的情况下为其增加额外功能，比如JDK的BufferedInputStream就是一个典型案例，它为输入流增加了缓冲功能。

装饰者模式包含如下四种角色：主体（Subject），实际主体（ConcreteSubject），装饰基类（Decorator），实际装饰类（ConcreteDecorator）。它们的类图如下

![blob.png](/assets/resources/image/20170709/1499584620894064749.png "1499584620894064749.png")

编写一个案例，为目标类增加一个功能，每当方法执行完后，将日志信息输出来。

```java
public interface Subject {
	void process();
	
	public static void main(String[] args) {
		Subject subject = new ConcreteDecorator(new ConcreteSubject());
		subject.process();
	}
}

class ConcreteSubject implements Subject {
	public void process() {
		System.out.println("执行处理-----");
	}
}

class Decorator implements Subject {
	protected Subject subject;

	public Decorator(Subject subject) {
		this.subject = subject;
	}

	public void process() {
		subject.process();
	}

}

class ConcreteDecorator extends Decorator {

	public ConcreteDecorator(Subject subject) {
		super(subject);
	}

	public void process() {
		subject.process();
		System.out.println("记录日志-----");
	}
}
```

看完这个例子，大家会不会觉得Decorator这个类没有起到实际作用。最初我也是这么认为的，但是在某些情况下，还是有它存在的必要。比如，当需要大量装饰类时，那么每个子类装饰类都需要实现所有的接口方法。可能造成子类装饰类代码有过多的重复。它和抽象类空实现有相似的作用。

代理模式和装饰者模式理解。

代理模式和装饰者模式的实现方式基本一样，但是它们的出发点不一致，代理模式主要目的是隐藏目标对象，并对其进行进一步操作；而装饰者模式主要目的是为了增强目标对象的功能。我认为代理模式其实是装饰者模式的一种特殊实现。

