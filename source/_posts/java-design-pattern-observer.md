---
title: Java 设计模式——观察者模式(Observer)
date: 2017-10-22 20:38:26
tags:
---
## 基本简介

观察者模式定义了对象之间一对多的依赖，这样一个对象状态改变了，他的依赖者就会收到通知并且自动更新

例如：通常数据改变了，界面会刷新数据显示区域，此时，数据就扮演了被观察者的角色，而界面扮演的是观察者的角色。再举一个例子：微信公众号是被观察者，微信用户是观察者，公众号更新了消息就会通知这些微信用户。

## 类图

![类图](/images/java-design-pattern/observer/diagram.jpg)

Subject: 抽象主题接口，对象使用这个接口注册为观察者

ConcreteSubject: 具体的主题接口

Observer: 所有观察者必须实现这个接口，如果主题更新了会调用 update() 方法

ConcreteObserver: 具体观察者

## 简单实现

**抽象观察者 ( Observer )**

> Observer.java

```
//抽象观察者
public interface Observer {
	void update(String message);
}
```

**抽象被观察者 ( Subject )**

> Subject.java

```
//抽象被观察者
public interface Subject {
	
	void attach(Observer observer);
	
	void detach(Observer observer);
	
	void subNotify();
}
```

**具体观察者 ( ConcrereObserver )**

> User.java

```
public class User implements Observer {

	private String name;
	
	public User(String name) {
		this.name = name;
	}
	
	@Override
	public void update(String message) {
		System.out.println("您好 " + name + " 我发布了一篇新的文章，请查看");
	}
	
}
```

**具体被观察者 ( ConcrereSubject )

> WeixinSubject.java

```
public class WeixinSubject implements Subject {
	
	private List<Observer> userList = new ArrayList<Observer>();

	@Override
	public void attach(Observer observer) {
		userList.add(observer);
	}

	@Override
	public void detach(Observer observer) {
		userList.remove(observer);
	}

	@Override
	public void subNotify() {
		for(Observer observer : userList) {
			observer.update("你好，我发布了一篇新文章");
		}
	}
	
}
```

**运行结果**：

![运行结果](/images/java-design-pattern/observer/result.png)


观察者将自己注册到被观察者的容器中时，被观察者不应该过问观察者的具体类型，而是应该使用观察者的接口。这样的优点是：假定程序中还有别的观察者，那么只要这个观察者也是相同的接口实现即可。一个被观察者可以对应多个观察者，当被观察者发生变化的时候，他可以将消息一一通知给所有的观察者。基于接口，而不是具体的实现——这一点为程序提供了更大的灵活性

## 优点

解除耦合，让耦合的双方都依赖于抽象，从而使得各自的变换都不会影响另一边的变换

## 缺点

在应用观察者模式时需要考虑一下开发效率和运行效率的问题，程序中包括一个被观察者、多个观察者，开发、调试等内容会比较复杂，而且在Java中消息的通知一般是顺序执行，那么一个观察者卡顿，会影响整体的执行效率，在这种情况下，一般会采用异步实现