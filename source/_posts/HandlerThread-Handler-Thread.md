---
title: 'HandlerThread, Handler, Thread, Runnable'
date: 2016-06-05 14:53:52
categories: Android
tags:
---

![img](../images/ckxt0.jpg)

Handler会关联一个单独的线程和消息队列。
Handler默认关联主线程，虽然要提供Runnable参数 ，但默认是直接调用Runnable中的run()方法。也就是默认下会在主线程执行，如果在这里面的操作会有阻塞，界面也会卡住。
如果要在其他线程执行，可以使用HandlerThread。
HandlerThread继承于Thread，所以它本质就是个Thread。与普通Thread的差别就在于，主要的作用是建立了一个线程，并且创立了消息队列，有来自己的looper,可以让我们在自己的线程中分发和处理消息。

<!--more-->

# HandlerThread的使用

```java
//Handler handler = new Handler() {
//...
//}
HandlerThread uIhandlerThread = new HandlerThread("update");
uIhandlerThread.start();
//Handler UIhandler = new Handler(uIhandlerThread.getLooper());
Handler uIhandler = new Handler(uIhandlerThread.getLooper(), new Callback() {
	public boolean handleMessage(Message msg) {
	Bundle b = msg.getData();
	int age = b.getInt("age");
	String name = b.getString("name");
	System.out.println("age is " + age + ", name is" + name);
	System.out.println("Handler--->" + Thread.currentThread().getId());
	System.out.println("handlerMessage");
	return true;
	}
});
```

当要停止uIhandlerThread执行时用：
```java
if (uIhandlerThread != null) {
	pointThread.quit();
}
```

# Handler的使用

目前常使用的有两种用法，
一种是自定义Handler，在handleMessage进行事件的处理， 这个Message可以是在其他线程中send的，或者在主线程中send。
在线程中发送信息到主进程：
## 1.定义handler

```java
public class MyHandler extends Handler {
	@Override
	public boolean handleMessage(Message msg) {
		switch (msg.what){
		case 1 :
		....
		}
	}
}
MyHandler myHandler = new MyHandler();
```

或者

```java
Handler myHandler = new Handler(new Callback() {
	// 参数也可以为（this.getMainLooper()，new Callback(){}）不写则默认为主进程的Looper
	@Override
	public boolean handleMessage(Message msg) {
		// TODO Auto-generated method stub
		return false;
	}
});
```

## 2.新建一个线程

```java
Thread sender = new Thread(){
	@Override
	public void run() {
		....
		//Message msg = new Message();
		Message msg = myHandler.obtainMessage(); //可以从handler中拿出message，省去了重新实例化的内存开销
		//myHandler.sendMessage(msg);
		msg.sendToTarget();
		//myHandler.sendEmptyMessage(intWhat);
	}
}
sender.start();
```

在主线程中发信息到handler
直接在主进程，不在线程中mHandler.sendMessage(msg);
另一种为post一个线程进去，执行线程。直到线程退出或者是handler被removeCallbacks。
定义一个线程Tread名为sender（不重复了）。
然后执行，myHandler.post(sender);
这样线程就在handler中执行。如果要停止线程的话：
```java
if(myHandler!=null) {
	myHandler.removeCallbacks(senderObj);
}
```

也可用一个Runnable来代替Thread

```java
Runnable r = new Runnable() {

	@Override
	public void run() {
		// TODO Auto-generated method stub
	}
};
myHandler.post(r);
```

以上两种，是否使用handler的post来启动，差别在与是否开启新线程来执行处理。
**使用post方法时，直接调用Thread或Runnable的run方法，所有处理都在主线程中进行，并没有开启定义的Thread或Runnable新的线程！！**

# 关于Thread和Runnable的区别

参考网址：http://www.360doc.com/content/10/1219/22/573136_79607619.shtml
Thread和Runnable是实现java多线程的两种方式，Thread是类，Runnable为接口，建议使用Runnable来实现多线程。
如果让一个线程实现Runnable接口，那么当调用这个线程的对象开启多个线程时，可以让这些线程调用同一个变量；
若这个线程是由继承Thread类而来，则要通过内部类来实现上述的功能，利用的就是内部类可任意访问外部类变量这个特性。（精辟！！）

## 实现Runnable接口

```java
public class ThreadTest {
	public static void main(String[] args) {
		MyThread mt=new MyThread();
		new Thread(mt).start(); //通过实现Runnable的类的对象来开辟第一个线程
		new Thread(mt).start(); //通过实现Runnable的类的对象来开辟第二个线程
		new Thread(mt).start(); //通过实现Runnable的类的对象来开辟第三个线程
		//由于这三个线程是通过同一个对象mt开辟的，所以run()里方法访问的是同一个index
	}
}
```
```java
//实现Runnable接口
class MyThread implements Runnable {
	int index=0;
	public void run() {
		for(;index<=200;)
		System.out.println(Thread.currentThread().getName()+":"+index++);
	}
}
```

## 继承Thread

```java
public class ThreadTest {
	public static void main(String[] args) {
		MyThread mt=new MyThread();
		mt.getThread().start(); //通过返回内部类的对象来开辟第一个线程
		mt.getThread().start(); //通过返回内部类的对象来开辟第二个线程
		mt.getThread().start(); //通过返回内部类的对象来开辟第三个线程
		//由于这三个线程是通过同一个匿名对象来开辟的，所以run()里方法访问的是同一个index
	}
}
```
```java
class MyThread {
	int index=0;
	//定义一个内部类，继承Thread
	private class InnerClass extends Thread {
		public void run() {
			for(;index<=200;)
			System.out.println(getName()+":"+index++);
		}
	}
	//这个函数的作用是返回InnerClass的一个匿名对象
	Thread getThread() {
		return new InnerClass();
	}
}
// 这里有一个问题：如果内部类要访问一个外部变量或方法，那么这个变量或方法必须定义为final，但为什么这里的变量index不用定义为final就可以被内部类访问？
```

# Thread的使用

```java
Thread sender = new Thread(){
	@Override
	public void run() {
		....
	}
}
sender.start();
```

线程的停止
```java
if (sender != null) {
	//sender.quit();
	sender.join(); // 执行完毕当前处理后停止线程
}
```