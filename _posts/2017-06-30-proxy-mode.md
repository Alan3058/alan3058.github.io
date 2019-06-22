---
layout: post
title: [代理模式]
categories: [DesignPattern]
tags: [DesignPattern,代理模式]
id: [1486120374239232]
fullview: false
---

代理模式是指为目标对象提供代理，以便控制目标对象的访问。比如在某些情况下，客户端不适合或者不能直接访问目标对象，这时就可以委托代理类作为一个中介者暴露给客户端。它的用途非常广泛，比如那些卖保险的人，他们其实是为公司代理卖保险，所以他们也被公司称为代理人。还有我们常说的代理商，代理服务器等，其实都是代理模式的思想。

代理模式主要包含以下三个角色:主体(Subject)，实际主体(ConcreteSubject)，代理主体(Proxy)。其类图如下

![blob.png](/assets/resources/image/20170708/1499530183474070743.png "1499530183474070743.png")

编写一个实例，由于ConcreteSubject实例构建太过于占用资源，故而使用代理类Proxy对象代替ConcreteSubject对象，实现只有在真正使用ConcreteSubject方法时才对其实例化。

```java
public interface Subject {
	void process();

	public static void main(String[] args) {
		Subject subject = new Proxy();
		subject.process();
	}
}

class ConcreteSubject implements Subject {
	public ConcreteSubject() {
		System.out.println("进行耗时的构建对象操作");
	}

	public void process() {
		System.out.println("处理-----------");
	}
}

class Proxy implements Subject {
	private Subject real;

	public Proxy() {
		System.out.println("创建代理对象");
	}

	public synchronized void process() {
		if (real == null) {
			real = new ConcreteSubject();
		}
		real.process();
	}
}
```

该案例我们称之为虚拟代理，除了虚拟代理还有远程代理（比如java的rmi）、访问代理（控制用户权限访问真实对象）、缓存代理（比如代理服务器）。

