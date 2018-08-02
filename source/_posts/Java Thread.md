---
title: Java Thread
date: 2018-07-27 10:33:28
tags:
---

##### 简单介绍一下线程

- 线程是一个程序的多个执行路径，执行调度的单位，依托于进程存在。 线程不仅可以共享进程的内存，而且还拥有一个属于自己的内存空间，这段内存空间也叫做线程栈，是在建立线程时由系统分配的，主要用来保存线程内部所使用的数据，如线程执行函数中所定义的变量。

<!--more-->

##### 如何开启线程（三种方法）

禁止直接调用 run 方法，否则不是另开线程，还是在原来的线程工作。

```java
class MyThread extend Thread {
    public void run {
        //继承线程
    }
}

MyThread myThread = new MyThread();
myThread.start();//开启线程
```

```java
class MyThread implement Runnable {
    public void run {
        //实现 Runnable 接口
    }
}

MyThread myThread = new MyThread();
Thread t = new Thread(myThread);
t.start();//开启线程
```

```java
new Thread(new Runnable(){
    public void run() {
        //匿名内部类实现线程开启
    }
}).start();
```

##### 线程中断的原因

1. 线程的 run 方法执行完最后一句语句后
2. 线程在 run 方法中出现没有捕获的异常
3. 可以通过 interrupt 方法来请求终止线程
4. 被中断的线程可以决定如何相应中断（中断线程只不过是引起线程的注意）

```java
void interrupt()//发送中断请求，线程中断状态设置为 true ，若线程被 sleep 阻塞，抛出 InterruptedException 异常。
static boolean interrupted()//测试线程是否被中断（无论如何，中断状态重置为 false）
boolean is Interrupted()//测试线程是否被终止（不改变线程的中断状态）
static Thread currentThread()//获取当前执行线程的 Thread 对象
```

##### 线程的状态（5 种状态）

![image](http://duqiblog.qiniudn.com/Java%E7%BA%BF%E7%A8%8B%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

- New （新建状态） 即： Thread thread = new Thread();

- Runnable （就绪状态） 其他线程调用 thread.start(); 等待 CPU 调度运行

- Running （运行状态） 线程只能从就绪状态进入运行状态

- Blocked （阻塞状态）放弃 CPU 运行。原因有三种  

  - 等待阻塞 ：thread.wait();  

  - 同步阻塞 ：synchronized 同步锁被其他线程占用  

  - 其他阻塞 ：调用线程的 sleep() 或 join() 或 发出I/O请求。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。 

- Dead （死亡状态）执行完 或 因异常 退出 run 方法。

##### 线程的属性

- 优先级 ：由于在 Windows 操作系统下有 7 个优先级别，但在 Linux 操作系统，线程的优先级别被忽略，所有线程具有相同的优先级别。所以要少用优先级别来构建程序功能，避免出现因线程优先级别出现的程序烦恼。

  ```java
  void setPriority(int newPriority); //1 - 10 之间
  static int MIN_PRIORITY //最小优先级别 1
  static int MAX_PRIORITY //最大优先级别 10
  static int NORM_PRIORITY //默认优先级别 5
  static void yield();//线程让步（对相同优先级别的线程）
  ```

- 守护线程 ：没啥用，就为其他线程提供服务。

- 未捕获异常处理器 setUncaughtExceptionHandler 方法为线程安装一个处理器。 setDefaultUncaughtExceptionHandler 为线程安装默认处理器。默认的处理器为空，替换处理器可以使用日志 API 来发送捕获异常的报告到日志文件中。  

##### 线程同步

为什么需要线程同步？  
答：由于 2 个或多个线程共享同一数据，导致数据错乱。（也称竞争条件）  

实现代码同步的方法：

- 锁对象 （ReentrantLock 保护代码块）

  ```java
  myLock.lock(); //一个 ReentrantLock 对象
  try {
      //被锁的代码块
  }finally {
      myLock.Unlock(); //一定要解锁，否则其他线程永远阻塞
  }
  ```

- 使用 synchronized 关键字（建议使用）

  ```java
  public sychronized void method (){
      //同步代码块（同步方法）
  }
  
          //等价于
  
  public void method(){
      this,intrinsicLock.lock();
      try {
          //同步代码
      } finally {
              this,intrinsicLock.unlock();
      }
  }
  ```

##### 带返回值的线程（Callable 和 FutureTask）以及 线程池

```java
class MyCallable implements Callable{ 
        private String oid; 

        MyCallable(String oid) { 
                this.oid = oid; 
        } 

        @Override 
        public Object call() throws Exception { 
                return oid+"任务返回的内容"; 
        } 
}

//创建一个线程池 
ExecutorService pool = Executors.newFixedThreadPool(2); 
//创建两个有返回值的任务 
Callable c1 = new MyCallable("A"); 
Callable c2 = new MyCallable("B"); 
//执行任务并获取Future对象 
Future f1 = pool.submit(c1); 
Future f2 = pool.submit(c2); 
//从Future对象上获取任务的返回值，并输出到控制台 
System.out.println(">>>"+f1.get().toString()); 
System.out.println(">>>"+f2.get().toString()); 
//关闭线程池 
pool.shutdown();
```
