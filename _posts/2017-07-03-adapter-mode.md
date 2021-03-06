---
layout: post
title: [适配器模式]
categories: [DesignPattern]
tags: [DesignPattern,适配器模式]
id: [1582111538544640]
fullview: false
---

适配器模式是指通过包装原对象，使得其转换成目标接口。现实生活中的电源适配器就是一个典型的案例，它将高电压转换成适应于对应电器的低电压，以便电器可以正常通电。

适配器模式包含以下三个角色:主体(Subject)，实际主体(ConcreteSubject)，适配主体(Adapter)，实际适配主体(ConcreteAdapter)。它们之间的类图关系如下

![blob.png](/assets/resources/image/20170708/1499531248110077393.png "1499531248110077393.png")

编写一个案例，假如目前已由高电压220V，手机最大承受电压12V，现需设置一个适配器，使得高电压可以用于充手机。

```java
public class Subject {
	public void operation0() {
		System.out.println("输出高电压220V");
	}
	
	public static void main(String[] args) {
		Adapter adapter = new ConcreteAdapter(new Subject());
		adapter.operation1();
	}

}

interface Adapter {
	void operation1();
}

class ConcreteAdapter implements Adapter {
	private Subject subject;
	
	public ConcreteAdapter(Subject subject){
		this.subject = subject;
	}

	public void operation1() {
		subject.operation0();
		System.out.println("转换低电压12V充电");
	}
}
```


