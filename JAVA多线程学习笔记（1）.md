## JAVA多线程学习笔记（1）

### 一、创建新进程

  由于这部分比较基础，这里不过多赘述，主要以代码呈现

#### 1、java.lang.Thread的extends

```java
public class MyThread extends Thread {
    public void run() {
        ...
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start();   //启动新进程，调用run()
        ...
    }
}
```

#### 2、java.lang中的Runnable接口

```java
public interface Runnable {
    public abstract void run();
}
```

```java
public class MyRunnable implements Runnable {
    public void run() {
        ...
    }
}
public class Main{
    public static void main(String[] args) {
        /*
         *different to Thread
         *创建MyRunnable的实例，再以该实例为参数创建Thread类的实例
        */
        new Thread(new MyRunnable()).start();      
        ...
    }
}
```

#### 3、java.util.concurrent.ThreadFactory中的线程创建

  java.util.concurrent包中包含抽象化ThreadFactory接口

```java
ThreadFactory factory = Executors.defaultThreadFactory();
//通过Executors.defaultThreadFactor获取ThreadFactory
factory.newThread(new myRunnable()).start()
```

#### 4、简单操作和概念简述

具体更加细节的使用会在后面内容中提到，在这里只是做一个简单的罗列和概括。

##### （1）Thread.sleep()

 线程睡眠 Thread.sleep(long millis) 转到阻塞状态

```java
try {
    Thread.sleep(10);
} catch (InterruptedException e) {
}
```

##### (2)Thread.yield() 

​       sleep 方法使当前运行中的线程睡眼一段时间，进入不可运行状态，且sleep 方法允许较低优先级的线程获得运行机会。

​       yield 方法使当前线程让出 CPU 占有权，但让出的时间是不可设定的。实际上，yield()方法对应了如下操作：先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把 CPU  的占有权交给此线程，否则，继续运行原来的线程。所以yield()方法称为“退让”，它把运行机会让给了同等优先级的其他线程。另外，yield()  方法执行时，当前线程仍处在可运行状态，所以，不可能让出较低优先级的线程些时获得 CPU 占有权。

##### (3)interrupt()

中断本线程

**本线程中断自己是被允许的，其它线程调用本线程的interrupt()方法时，会通过checkAccess()检查权限。这有可能抛出SecurityException异常。

**如果本线程是处于阻塞状态：调用线程的wait(), wait(long)或wait(long, int)会让它进入等待(阻塞)状态，或者调用线程的join(), join(long), join(long, int), sleep(long), sleep(long, int)也会让它进入阻塞状态。

**若线程在阻塞状态时，调用了它的interrupt()方法，那么它的“中断状态”会被清除并且会收到一个InterruptedException异常。例如，线程通过wait()进入阻塞状态，此时通过interrupt()中断该线程；调用interrupt()会立即将线程的中断标记设为“true”，但是由于线程处于阻塞状态，所以该“中断标记”会立即被清除为“false”，同时，会产生一个InterruptedException的异常。如果线程被阻塞在一个Selector选择器中，那么通过interrupt()中断它时；线程的中断标记会被设置为true，并且它会立即从选择操作中返回。

**如果不属于前面所说的情况，那么通过interrupt()中断线程时，它的中断标记会被设置为“true”。
中断一个“已终止的线程”不会产生任何操作。

**！！！！interrupt()并不会终止处于“运行状态”的线程！它会将线程的中断标记设为true。**

##### (4)synchronized方法（同步方法）

```java
synchronized void method() {
    ...
}
// is equal to 
void method() {
    synchronized(this) {
        ...         //synchronized静态方法和动态方法在此例上有所区别，具体请自己查阅资料
    }
}
```

声明一个方法，代表这个方法只能由一个线程运行

一个实例中的synchronized方法每次只能由一个线程运行，而非synchronized则可以由两个以上线程运行。

一个线程运行一个synchronized方法，则该进程获得了锁，其他进程就无法运行该方法，当该方法完成了，便会释放锁，一个一直等待锁的进程便会获得该锁，开始运行synchronized方法。

而非synchronized方法完全不受锁的影响，可以自由进入。

**注意：每个实例都拥有一个独立的锁**

##### (5)synchronized代码块

```java
synchronized(expression) {
    ...
}
```

##### (6)等待队列

所有实例都拥有一个等待队列，它是在wait方法执行后停止操作的线程的队列

当由其他线程的notify()或notifyAll()方法唤醒，或有其他线程的interrupt()方法唤醒，或wait方法超时时，退出等待队列。

##### (7)wait

让线程进入等待队列

```java
obj.wait();
```

**若要执行wait方法，进程必须持有锁**

##### (8)notify

从等待队列中取出一个线程（随机的）

**线程必须持有要调用的实例的锁**

##### (9)notifyAll

从等待队列中取出**所有**线程

**wait、notify、notifyAll是Object类的方法，也是Thread类的方法**

##### (10)join

等待线程终止

##### (11)setPriority() 、getPriority()



### 二、Single Threaded Execution模式

#### 1、SharedResource

在Single Threaded Execution模式中使用到了SharedResource作用的类

SharedResource可被多个线程访问，主要分为safeMethod和unsafeMethod，该模式就是为了保护unsafeMethod，使其同时只能由一个线程访问。

#### 2、何时使用

##### （1）多线程程序

​	单线程显然没必要使用，并且调用synchronized方法比一般方法花费时间更多，会降低程序性能。

​	引用《图解Java多线程设计模式》中一个不太恰当的例子：在单线程程序中使用synchronized好比一个独居在家的人上厕所关门一样。

##### （2）SharedResource可能被多个线程访问时

##### （3）SharedResource的状态会发生变化

​	例如，Immutable模式就完全没必要使用

##### （4）需要确保安全性时

​        java的集合类大多非线程安全，请考虑

​	可以参考java.util.Collections的API文档

#### 3、性能考虑

程序设计性能还是不得不考虑的一个环节

该模式主要在两方面会降低程序性能

##### （1）获取锁的花费

##### （2）线程conflict

  一个小tips：

​	java.util.Hashtable和java.util.concurrent.ConcurrentHashMap二者功能相似，都是线程安全，其中Hashtable采用Single Threaded Execution模式，更易发生线程冲突，ConcurrentHashMap不易。

​	

​	





