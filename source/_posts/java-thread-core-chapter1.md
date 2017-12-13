---
title: 学习笔记：Java 多线程核心技术（第一章）
date: 2017-09-25 16:53:02
tags:
---
# Chapter 1 多线程技能

## 使用多线程

使用多线程有两种方式：

* 继承 `Thread` 类重写 `run()` 方法

* 实现 `Runnable` 接口

首先来看第一种方法

```
package com.syn.thread;

public class MyThread extends Thread{
	
	@Override
	public void run() {
		super.run();
		System.out.println("MyThread");
	}
	
	public static void main(String []args) {
		MyThread t = new MyThread();
		t.start();
	}
}

```


其实，通过查看 `Thread` 类的源码可以知道这个类也是实现了 `Runnable` 接口，也就是说我们无论用哪种方法，都是直接或者间接的实现了 `Runnable` 接口

我们知道 Java 中只支持单继承的，如果某个程序需要继承某个类，并且需要使用线程，那么第一种方法就没有办法实现了

但是好在我们可以使用第二种方法来实现 `Runnable` 接口

```
package com.syn.thread;

public class MyThread implements Runnable {
	
	@Override
	public void run() {
		System.out.println("MyThread");
	}
	
	public static void main(String []args) {
		MyThread t = new MyThread();
		Thread th = new Thread(t);
		th.start();
	}
}
```

在上面的例子中都是使用 `xx.start()` 方法来启动线程，而不是直接调用 `xx.run()`

`Thread.java` 类中的 `start()` 方法会通知 **线程规划器**，并告知此线程已经准备就绪，等待调用对象的 `run()` 方法，也就是让系统安排一个时间来调用 `Thread` 中的 `run()` 方法，此时是异步运行，`main` 方法中的代码可以一起运行。如果是直接调用 `xx.run()` 方法，那么就不是异步执行， 而是同步执行。同步执行只有在运行完 `run()` 方法之后才能执行后面的代码，并且不会交给 **线程规划器** 来处理的

## 实例变量与线程安全

实例变量与线程安全有两种情况，第一是 *不共享数据* 第二种是*共享数据*

不共享数据就是变量不共享，也就不存在多个线程访问同一个实例变量的情况

共享数据的情况就相对麻烦了，因为会有多个线程访问同一个实例变量导致数据不正确

```
package com.syn.thread;

public class MyThread extends Thread {
	private int count = 5;
	
	@Override
	public void run() {
		super.run();
		count--;
		System.out.println("由 " + this.currentThread().getName() + " 计算 count " + count);
	}
	
	public static void main(String []args) {
		MyThread t1 = new MyThread();
		Thread a = new Thread(t1,"A");
		Thread b = new Thread(t1,"B");
		Thread c = new Thread(t1,"C");
		Thread d = new Thread(t1,"D");
		Thread e = new Thread(t1,"E");
		a.start();
		b.start();
		c.start();
		d.start();
		e.start();
	}
}
```

得出的结果是
> 由 A 计算 count 4

> 由 B 计算 count 2

> 由 C 计算 count 2

> 由 D 计算 count 1

> 由 E 计算 count 0


这样我们发现，B 和 C 打印出来的 count 值都是 2，说明了 B 和 C 线程是同时对 count 进行处理，这就产生了**非线程安全**的问题

解决非线程安全的问题是使用 `synchronized` 关键字，代码更改如下：

```
package com.syn.thread;

public class MyThread extends Thread {
	private int count = 5;
	
	@Override
	synchronized public void run() {
		super.run();
		count--;
		System.out.println("由 " + this.currentThread().getName() + " 计算 count " + count);
	}
	
	public static void main(String []args) {
		MyThread t1 = new MyThread();
		Thread a = new Thread(t1,"A");
		Thread b = new Thread(t1,"B");
		Thread c = new Thread(t1,"C");
		Thread d = new Thread(t1,"D");
		Thread e = new Thread(t1,"E");
		a.start();
		b.start();
		c.start();
		d.start();
		e.start();
	}
}
```
这次的结果就不会出现值一样的问题，原因是：

在 `run()` 方法前加入 `synchronized` 关键字，第一个线程执行的时候获得了锁，其他的线程即将执行会检查是否上锁，如果第一个线程未释放锁，就会等待直到第一个线程释放之后才能得到这个锁

**非线程安全** 是指多个线程对同一个对象中的同一个实例变量进行操作时会出现值被更改、值不同步的情况

***

## 线程的常用方法

**currentThread()方法**

可返回代码段正在被哪个线程调用执行

**isAlive()方法**

这个方法可判断当前线程是否处于**活动状态**

**活动状态**是线程已经启动但是尚未终止，如果线程处于正在运行或者准备的状态，就认为线程是“存活”的

**sleep()方法**

在指定毫秒数中让当前正在执行的线程休眠

**getId()方法**

这个方法是取得线程唯一标识符

## 如何停止线程

停止线程有三种方法停止线程

* 使用退出标志让线程正常退出
* 使用 `stop` 方法强行终止线程
* 使用 `interrupt` 方法中断线程

第一种方法就是使用退出标志，正常执行完退出，这里不做赘述

第二种方法使用 `stop` 方法强行退出是非常暴力的，如果使用这个方法停止线程可能会导致某些需要清理性的工作不能完成，也可能会导致对锁定对象进行解锁，导致数据不一致的问题

第三方法是比较常用的，这个方法不会终止一个正在运行的线程，需要加入一个判断才能完成线程的停止

我们会想到 `interrupted()` 方法和 `isInterrupted()` 方法，这两个方法有什么不同？我们来一探究竟

首先 `interrupted()` 方法是**测试当前线程**是否已经中断，而 `isInterrupted()` 方法是**测试线程**是否已经中断

`interrupted()` 方法在一次执行后具有将状态标志清除为 false 的状态。即连续两次调用该方法后，第二次调用则会返回 false

前面说了这个方法不会终止一个正在运行的线程而是需要加入一个判断来停止。这样做的原因是，Java 的中断并不是真正的中断线程，而只设置标志位（中断位）来通知用户。如果你捕获到中断异常，说明当前线程已经被中断，也就是说不需要继续保持中断位了

## 暂停线程

被弃用的两个方法：`suspend()`、`resume()`

这两个方法一个是用来暂停线程，一个是用来恢复线程

为什么要弃用他们呢？

* 缺点一：独占。如果使用不当，会造成公共的同步对象独占，使得其他进程无法访问公共同步对象

如果某个线程占用了某个对象并且让它暂停，并且永远不恢复，导致其他的线程无法访问这个被占用的对象，于此相类似的还有独占锁的情况。

* 缺点二：不同步。使用这两个方法容易出现因为线程暂停而导致数据不同步的情况

## 线程的优先级

优先级较高的线程将得到更多的 CPU 资源

设置线程优先级有助于帮助 “线程规划器” 确定下一次选择哪一个线程来执行

线程优先级分为 0-10 个等级，如果小于 1 或者大于 10，会抛出 `IllegalArgumentException` 异常

**优先级具有继承性**：比如线程 A 启动线程 B，则 B 线程的优先级与 A 是一样的

**优先级具有规则性**：高优先级的线程总是大部分先执行完，但是不代表高优先级的线程全部先执行完

**优先级具有随机性**：优先级搞的线程不一定每次都先执行完

## 守护线程

Java 中有两种线程，一种是用户线程，一种是守护线程

守护线程是一种特殊的线程，当进程中没有非守护线程了那么守护线程就会自动销毁

书中有一个很形象的例子：守护线程相当于保姆，如果幼儿园里没有小朋友了（也就是说没有非守护线程了）那么保姆就不需要干活儿（守护线程自动销毁）；如果还有最后一个小朋友，保姆也得干活（守护线程不销毁）

## 总结

这一章的内容算是对线程基础知识的回顾，在各种情况发生时有许多意想不到的地方，这就是考验基础的时候了。

笔记中只是少部分的记录了一下，各种情况需要翻阅书籍继续学习才能体会其中的奥妙。