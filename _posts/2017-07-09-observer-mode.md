---
layout: post
title: [观察者模式]
categories: [DesignPattern]
tags: [DesignPattern,观察者模式]
id: [2224026193756160]
fullview: false
---

观察者模式

```java
import java.util.ArrayList;
import java.util.Collection;

public interface Subject {
	void process();
	void addObserver(Observer e);
	void delObserver(Observer e);
	void notifyObserver();

	public static void main(String[] args) {
		Subject subject = new ConcreteSubject();
		subject.addObserver(new ConcreteObserver());
		subject.process();
	}
}

class ConcreteSubject implements Subject {
	
	private Collection<Observer> elements = new ArrayList<Observer>();
	
	public void process() {
		System.out.println("改变信息-----------");
		System.out.println("准备发送通知-----------");
		notifyObserver();
	}

	public void addObserver(Observer e) {
		elements.add(e);
	}

	public void delObserver(Observer e) {
		elements.remove(e);
	}

	public void notifyObserver() {
		for(Observer observer : elements){
			observer.update();
		}
	}
}

interface Observer{
	void update();
}

class ConcreteObserver implements Observer {
	public void update() {
		System.out.println("收到通知--------");
	}
}
```


