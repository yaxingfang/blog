---
title: Java八股文-Java并发
copyright: true
mathjax: false
categories:
  - Java八股文
date: 2023-01-23 15:29:48
tags:
toc: true
urlname: java-concurrent-programming
---

> 整理的Java并发相关知识点和面试题，部分内容摘自网络，如有侵权请联系我～	<!--more-->

## 基础知识

### 并发编程的优缺点

优点：为了能提高程序的执行效率，提高程序运行速度；

缺点：存在内存泄漏、上下文切换、线程安全、死锁等问题。

#### 并发编程三要素是什么？在 Java 程序中怎么保证多线程的运行安全？

并发编程三要素（线程的安全性问题体现在）：

**原子性**：原子，即一个不可再被分割的颗粒。原子性指的是一个或多个操作要么全部执行成功要么全部执行失败。（synchronized，Lock）（线程切换引起的原子性问题）

**可见性**：一个线程对共享变量的修改，另一个线程能够立刻看到。（synchronized，Lock，volatile）（JMM 内存模型导致的可见性问题）

**有序性**：程序执行的顺序按照代码的先后顺序执行。（synchronized，Lock，volatile）（指令重排序带来的有序性问题）

> volatile不能保证并发安全，比如多线程对一个volatile的int变量进行加1操作，最终得到的数字可能比预期小，就是因为++操作不是原子性的，而volatile也不能保证原子性，所以就会有这个问题，可以使用`atomicInteger.getAndSet()`原子操作。

### 线程和进程

#### 进程间的通信方式

1. **管道/匿名管道(Pipes)** ：用于具有亲缘关系的父子进程间或者兄弟进程之间的通信；
2. **有名管道(Names Pipes)**：匿名管道由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提出了有名管道。有名管道严格遵循**先进先出(first in first out)**。有名管道以磁盘文件的方式存在，可以实现本机任意两个进程通信；
3. **信号(Signal)** ：信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生；
4. **消息队列(Message Queuing)** ：消息队列是消息的链表，具有特定的格式，存放在内存中并由消息队列标识符标识。**消息队列克服了信号承载信息量少，管道只能承载无格式字节流以及缓冲区大小受限等缺点**；
5. **信号量(Semaphores)** ：信号量是一个计数器，用于多进程对共享数据的访问，信号量的意图在于进程间同步。这种通信方式主要用于解决与同步相关的问题并避免竞争条件。  
6. **共享内存(Shared memory)** ：使得多个进程可以访问同一块内存空间，不同进程可以及时看到对方进程中对共享内存中数据的更新。这种方式需要依靠某种同步操作，如互斥锁和信号量等。可以说这是最有用的进程间通信方式。
7. **套接字(Sockets)**：此方法主要用于在客户端和服务器之间通过网络进行通信。套接字是支持 TCP/IP 的网络通信的基本操作单元，可以看做是不同主机之间的进程进行双向通信的端点，简单的说就是通信的两方的一种约定，用套接字中的相关函数来完成通信过程。

#### 线程间的同步方式

线程同步是两个或多个共享关键资源的线程的并发执行。应该同步线程以避免关键的资源使用冲突。

1. **互斥量(Mutex)**：采用互斥对象机制，只有拥有互斥对象的线程才有访问公共资源的权限。因为互斥对象只有一个，所以可以保证公共资源不会被多个线程同时访问，比如 Java 中的 synchronized 关键词和各种 Lock 都是这种机制；
2. **信号量(Semphares)** ：它允许同一时刻多个线程访问同一资源，但是需要控制同一时刻访问此资源的最大线程数量；
3. **事件(Event)** ：wait/notify：通过通知操作的方式来保持多线程同步，还可以方便的实现多线程优先级的比较操作。

#### 什么是线程和进程?

**进程**：进程是应用程序的一次运行，每个进程都有自己独立的内存空间；是操作系统资源分配的基本单位；

**线程**：线程是处理器调度和执行的基本单位，一个进程中可以有多个线程，线程共享进程的内存空间和资源。

#### 什么是上下文切换?

多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。

当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换**。

上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。

#### 守护线程和用户线程有什么区别呢？

- **用户 (User) 线程**：运行在前台，执行具体的任务
- **守护 (Daemon) 线程**：运行在后台，为其他前台线程服务。一旦所有用户线程都结束运行，守护线程会随 JVM 一起结束工作

**注意事项：**

1. `setDaemon(true)`必须在`start()`方法前执行，否则会抛出 `IllegalThreadStateException` 异常
2. 在守护线程中产生的新线程也是守护线程
3. 不是所有的任务都可以分配给守护线程来执行，比如读写操作或者计算逻辑
4. 守护 (Daemon) 线程中不能依靠 finally 块的内容来确保执行关闭或清理资源的逻辑。因为我们上面也说过了一旦所有用户线程都结束运行，守护线程会随 JVM 一起结束工作，所以守护 (Daemon) 线程中的 finally 语句块可能无法被执行。

#### 如何在 Windows 和 Linux 上查找哪个线程cpu利用率最高？

Windows上面用任务管理器看，Linux下可以用 top 这个命令看。

#### 什么是线程死锁

死锁是指两个或两个以上的进程（线程）在执行过程中，由于竞争资源造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。

如下图所示，线程 A 持有资源 2，线程 B 持有资源 1，他们同时都想申请对方的资源，所以这两个线程就会互相等待而进入死锁状态。

![线程死锁](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/202206152131355.png)

下面通过一个例子来说明线程死锁，代码模拟了上图的死锁的情况 (代码来源于《并发编程之美》)：

```java
public class DeadLockDemo {
    private static Object resource1 = new Object();	//资源 1
    private static Object resource2 = new Object();	//资源 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}
```

输出结果

```java
Thread[线程 1,5,main]get resource1
Thread[线程 2,5,main]get resource2
Thread[线程 1,5,main]waiting get resource2
Thread[线程 2,5,main]waiting get resource1
```

线程 A 通过 synchronized (resource1) 获得 resource1 的监视器锁，然后通过`Thread.sleep(1000)`；让线程 A 休眠 1s 为的是让线程 B 得到CPU执行权，然后获取到 resource2 的监视器锁。线程 A 和线程 B 休眠结束了都开始企图请求获取对方的资源，然后这两个线程就会陷入互相等待的状态，这也就产生了死锁。上面的例子符合产生死锁的四个必要条件。

#### 形成死锁的四个必要条件是什么

1. 互斥条件：一个资源只能被一个线程占用，直到被该线程释放；
2. 请求与保持条件：一个线程因请求被占用资源而发生阻塞时，对已获得的资源保持不放；
3. 不剥夺条件：线程；已获得的资源在末使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源；
4. 循环等待条件：当发生死锁时，所等待的线程必定会形成一个环路（类似于死循环），造成永久阻塞

#### 如何避免线程死锁

我们只要破坏产生死锁的四个条件中的其中一个就可以了。

**破坏互斥条件**：无法破坏，因为我们用锁本来就是想实现互斥访问临界资源；

**破坏请求与保持条件**：一次性申请所有的资源；

**破坏不剥夺条件**：占用部分资源的线程进一步申请其他资源时，**如果申请不到，可以主动释放它占有的资源**；

**破坏循环等待条件**：靠**按序申请**资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。

我们对线程 2 的代码修改成下面这样就不会产生死锁了。

```java
new Thread(() -> {
    synchronized (resource1) {
        System.out.println(Thread.currentThread() + "get resource1");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread() + "waiting get resource2");
        synchronized (resource2) {
            System.out.println(Thread.currentThread() + "get resource2");
        }
    }
}, "线程 2").start();
```

输出结果

```java
Thread[线程 1,5,main]get resource1
Thread[线程 1,5,main]waiting get resource2
Thread[线程 1,5,main]get resource2
Thread[线程 2,5,main]get resource1
Thread[线程 2,5,main]waiting get resource2
Thread[线程 2,5,main]get resource2
```

我们分析一下上面的代码为什么避免了死锁的发生?

线程 1 首先获得到 resource1 的监视器锁，这时候线程 2 就获取不到了。然后线程 1 再去获取 resource2 的监视器锁，可以获取到。然后线程 1 释放了对 resource1、resource2 的监视器锁的占用，线程 2 获取到就可以执行了。这样就破坏了破坏循环等待条件，因此避免了死锁。

#### 创建线程有哪几种方式？

创建线程有四种方式：

- 继承 Thread 类；
- 实现 Runnable 接口；
- 实现 Callable 接口；
- 使用 Executors 工具类创建线程池

**继承 Thread 类**

1. 定义一个 Thread 类的子类，重写 run 方法
2. 创建自定义的线程子类对象
3. 调用子类实例的 start() 方法来启动线程

```java
public class Main {
    static class MyThread extends Thread {

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " run()方法正在执行...");
        }

    }
    public static void main(String[] args) {
        new MyThread().start();
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " main()方法执行结束");
    }
}
```

**实现 Runnable 接口**

1. 定义 Runnable 接口实现类 MyRunnable，并重写 run() 方法
2. 创建 MyRunnable 实例 myRunnable，以 myRunnable 作为 target 创建 Thread 对象，**该Thread对象才是真正的线程对象**
3. 调用线程对象的 start() 方法

```java
public class Main {
    static class MyRunnable implements Runnable {

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " run()方法正在执行...");
        }

    }
    public static void main(String[] args) {
        new Thread(new MyRunnable()).start();
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " main()方法执行结束");
    }
}
```

**实现 Callable 接口**

1. 创建实现 Callable 接口的类 myCallable
2. ⭐以 myCallable 为参数创建 FutureTask 对象
3. 将 FutureTask 作为参数创建 Thread 对象
4. 调用线程对象的 start() 方法

```java
public class Main {
    static class MyCallable implements Callable<Integer> {

        @Override
        public Integer call() throws Exception {
            System.out.println(Thread.currentThread().getName() + " call()方法执行中...");
            return 1;
        }
    }

    public static void main(String[] args) {
        FutureTask<Integer> futureTask = new FutureTask<>(new MyCallable());
        new Thread(futureTask).start();
        try {
            Thread.sleep(10);
            System.out.println(futureTask.get());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " main()方法执行完成");
    }
}
```

**使用 Executors 工具类创建线程池**

Executors 提供了一系列工厂方法用于创建线程池，返回的线程池都实现了 ExecutorService 接口。

主要有 new[Single/Fixed/Cached/Scheduled]ThreadPool 这四种线程池

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " run()方法执行中...");
    }
}

public class SingleThreadExecutorTest {n
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        MyRunnable myRunnable = new MyRunnable();
        for (int i = 0; i < 5; i++) {
            executorService.execute(myRunnable);
        }

        System.out.println("线程任务开始执行");
        executorService.shutdown();
    }
}
```

#### 说一下 runnable 和 callable 有什么区别？

相同点

- 都是接口
- 都可以用来实现多线程
- 都采用 Thread.start() 启动线程

**主要区别**

- Runnable 接口 run 方法无返回值；Callable 接口 call 方法有返回值，和 FutureTask 配合可以用来获取异步执行的结果
- Runnable 接口 run 方法无法捕获并处理异常；Callable 接口 call 方法可以捕获并处理异常

#### 线程的 run()和 start()有什么区别？

> start() 方法来启动一个线程，真正实现了多线程运行。调用 start() 方法无需等待run方法体代码执行完毕，可以直接继续执行其他的代码； 此时线程是处于就绪状态，并没有运行。 然后通过此 Thread 类调用方法 run() 来完成其运行状态， run() 方法运行结束， 此线程终止。然后 CPU 再调度其它线程。

start() 方法用于启动线程，run() 方法用于执行线程的运行时代码，直接调用run()，其实就相当于是调用了一个普通函数而已。
start() 只能调用一次，run() 可以重复调用。

#### 什么是 Callable 和 Future?

```java
FutureTask<Integer> futureTask = new FutureTask<>(new MyCallable());
new Thread(futureTask).start();
```

将 callable 实例传入 future，然后将 future 实例传入 Thread 创建线程，
之后可以用 future 来获取 callable  中 call() 的返回结果。

#### 线程的状态和基本操作

<img src="https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes1650518197279.png" alt="线程的状态" style="zoom:67%;" />

#### Java 中用到的线程调度算法是什么？

线程调度是指按照特定机制为多个线程分配 CPU 的使用权。

有两种调度模型：**时间片轮转**模型和**优先级调度**模型。

**Java虚拟机采用优先级调度模型**，是指优先让可运行池中优先级高的线程占用CPU，如果可运行池中的线程优先级相同，那么就随机选择一个线程，使其占用CPU。处于运行状态的线程会一直运行，直至它不得不放弃 CPU。

#### 请说出与线程同步以及线程调度相关的方法。

（1）wait()：使一个线程处于等待状态，并且释放所持有的对象的锁；

（2）sleep()：使一个正在运行的线程处于睡眠状态，是一个静态方法；

（3）notify()：唤醒一个处于等待状态的线程，当然在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由 JVM 确定唤醒哪个线程，而且与优先级无关；

（4）notityAll()：唤醒所有处于等待状态的线程，该方法并不是将对象的锁给所有线程，而是让它们竞争，只有获得锁的线程才能进入就绪状态；

#### wait() 和 sleep() 有什么区别？

两者都可以暂停线程的执行

- 类的不同：wait() 是 Object 类的方法，sleep() 是 Thread 线程类的静态方法。
- 释放锁：wait() 释放锁，sleep() 不释放锁。
- 用途不同：wait 通常被用于线程间交互/通信，sleep 通常被用于暂停执行。
- 自动苏醒：wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify() 或者 notifyAll() 方法。sleep() 方法执行完成后，线程会自动苏醒。或者可以使用wait(long timeout)超时后线程会自动苏醒。

#### 你是如何调用 wait() 方法的？使用 if 块还是循环？为什么？

> 使用 while 判断条件是否得到满足

使用 if 来判断会存在以下问题：

1. 另一个线程可能已经被唤醒并改变了条件状态。例如 notifyAll 会唤醒多个等待的线程。

2. 存在“伪唤醒”的情况，即在没有通知的情况下，线程也可能会苏醒过来，而此时是不应该唤醒的。

	```java
	synchronized (monitor) {
	    //  判断条件谓词是否得到满足
	    while(!locked) {
	        //  等待唤醒
	        monitor.wait();
	    }
	    //  处理其他的业务逻辑
	}
	```

#### 为什么线程通信的方法 wait()，notify() 和 notifyAll() 被定义在 Object 类里？

Java 中，任何对象都可以作为锁，并且线程通信的方法 wait()，notify() 等方法用于等待对象的锁或者是唤醒线程，那么要找一个可供任何对象使用的锁，因此将这些方法定义在 Object 中，Object 是所有类的父类。

#### Thread 类中的 yield 方法有什么作用？

使当前线程从运行状态变为就绪状态。

#### 线程的 sleep()方法和 yield()方法有什么区别？

1. sleep() 方法给其他线程运行机会时不考虑线程的优先级，因此会给低优先级的线程以运行的机会；

	yield()方法只会给**相同或更高优先级**的线程以运行的机会；

2. 线程执行 sleep() 方法后转入等待（waiting）状态，而执行 yield() 方法后转入就绪（ready）状态；

#### 如何停止一个正在运行的线程？

1. 当 run 方法完成后线程终止；
2. 使用 interrupt 方法中断线程。

#### notify() 和 notifyAll() 有什么区别？

如果线程调用了对象的 wait() 方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁。

notifyAll() 会唤醒所有的线程，notify() 只会唤醒一个线程。

notifyAll() 调用后，会将全部线程由等待池移到锁池，然后参与锁的竞争，竞争成功则继续执行，如果不成功则留在锁池等待锁被释放后再次参与竞争。而 notify()只会唤醒一个线程，具体唤醒哪一个线程由虚拟机控制。

#### 如何在两个线程间共享数据？

Java 线程之间的通信由 Java 内存模型（简称JMM）控制：

1. 所有的共享变量都存在主内存中
2. 每个线程都保存了一份该线程使用到的共享变量的副本
3. 如果线程A与线程B之间要通信：
	1. 线程A将本地内存A中更新过的共享变量刷新到主内存中去
	2. 线程B到主内存中去读取线程A之前已经更新过的共享变量。

#### Java 如何实现多线程之间的通讯和协作？

Java中线程通信协作的最常见的两种方式：

1. synchronized 加锁的线程 + Object 类的 wait()/notify()/notifyAll()

2. ReentrantLock 类加锁的线程 + Condition 类的 await()/signal()/signalAll()

#### 同步方法和同步块，哪个是更好的选择？

同步的范围越小越好。

因此，同步块是更好的选择，因为它不会锁住整个对象，而同步方法会锁住整个对象。

#### 什么是线程同步和线程互斥，有哪几种实现方式？

线程同步是两个或多个共享关键资源的线程的并发执行。应该同步线程以避免关键的资源使用冲突。

线程互斥是对某一共享资源而言，任何时刻最多只允许一个线程去使用，其它要使用该资源的线程必须等待，直到占用资源者释放该资源。

实现线程同步的方法

- 同步代码方法 / 方法块：sychronized 关键字修饰的方法 / 代码块
- 使用特殊变量域 volatile 实现线程同步：volatile关键字为域变量的访问提供了一种免锁机制
- 使用重入锁实现线程同步：reentrantlock 类是可重入、互斥、实现了 lock 接口的锁，与 sychronized 方法具有相同的基本行为和语义

#### 在监视器(Monitor)内部，是如何做线程同步的？

在 Java 虚拟机中，每个对象关联一个**监视器**，为了实现监视器的互斥功能，**每个对象都关联着一把锁**。

一旦方法或者代码块被 **synchronized** 修饰，那么这个部分就放入了监视器的监视区域，**确保一次只能有一个线程执行该部分的代码**，线程在获取锁之前不允许执行该部分的代码

另外 Java 还提供了显式监视器( Lock )和隐式监视器( synchronized )两种锁方案

#### Java 线程数过多会造成什么问题？

- 消耗过多的 CPU 资源

	如果可运行的线程数量多于可用处理器的数量，那么有线程将会被闲置。大量空闲的线程会占用许多内存，给垃圾回收器带来压力。

- 降低 JVM 稳定性

	在可创建线程的数量上存在一个限制，这个限制值将随着平台的不同而不同，并且承受着多个因素制约，包括 JVM 的启动参数、Thread 构造函数中请求栈的大小，以及底层操作系统对线程的限制等。如果破坏了这些限制，那么可能抛出 OutOfMemoryError 异常。

## 并发关键字

### synchronized

#### synchronized 的作用？

在 Java 中，synchronized 关键字是用来控制线程同步的，就是在多线程的环境下，控制 synchronized 代码段不被多个线程同时执行。synchronized 可以修饰静态方法、实例方法、代码块。

#### 怎么使用 synchronized 关键字

**synchronized关键字最主要的三种使用方式：**

- **修饰实例方法：** 给当前对象实例加锁；
- **修饰静态方法：** 给当前类加锁；
- **修饰代码块：**给指定对象加锁，进入同步代码块前要获得指定对象的锁。

**总结：** synchronized 关键字加到 static 静态方法和 synchronized(xxx.class) 代码块上都是是给 Class 类上锁。synchronized 关键字加到实例方法上是给对象实例上锁。

#### 双重校验锁实现对象单例（线程安全）

```java
public class Singleton {
    private volatile static Singleton uniqueInstance;

    private Singleton() {}

    public static Singleton getInstance() {
        // 第一次假如线程1，线程2，线程3到达这，都判断到null未实例化，加这个判断是为了让除了第一次实例化之后的其他线程判断到非空表明已经实例化过了，直接返回单例
        if (uniqueInstance == null) {	// 线程1、2、3有可能都进来了
            //类对象加锁
            synchronized (Singleton.class) {	// 如果多个线程都判断到未实例化，那么只会有一个线程锁住类并进行实例化
                if (uniqueInstance == null) {	// 如果不加这个判断，线程1拿到锁进行实例化之后，线程2拿到锁，进来直接进行实例化，这就产生多次实例化操作。如果加这个判断，线程2拿到锁之后，判断到已经实例化了，就不会再进行实例化了。
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }

}
```

另外，需要注意 instance采用 volatile 关键字修饰也是很有必要。

instance 采用 volatile 关键字修饰也是很有必要的，instance = new Singleton() 这段代码其实是分为三步执行：

1. 为 instance 分配内存空间
2. 初始化 instance
3. 将 instance 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getInstance() 后发现 instance不为空，因此**返回 T1 实例化但未初始化**的 instance。

#### synchronized 底层实现原理？（监视器monitor）

synchronized 是 Java 中的一个关键字，通过 javap 命令，查看相应的字节码文件。

synchronized 同步语句块的情况

```java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("synchronized 代码块");
        }
    }
}
```

通过JDK 反汇编指令 javap -c -v SynchronizedDemo

<img src="https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/image-20220912212554623.png" alt="monitorenter&monitorexit" style="zoom:50%;" />

可以看出在执行同步代码块之前之后都有一个 monitor 字样，其中前面的是 monitorenter，后面的是离开monitorexit，不难想象一个线程执行同步代码块，首先要获取锁，而获取锁的过程就是 monitorenter ，在执行完代码块之后，要释放锁，释放锁就是执行 monitorexit 指令。

为什么会有两个monitorexit呢？（line 19）

这个主要是防止在同步代码块中线程因异常退出，而锁没有得到释放，这必然会造成死锁（等待的线程永远获取不到锁）。**因此最后一个monitorexit是保证在异常情况下，锁也可以得到释放，避免死锁。**

#### synchronized 可重入的原理

重入锁是指一个线程获取到该锁之后，该线程可以继续获得该锁。
底层原理维护一个**计数器**，当线程获取该锁时，计数器加一，再次获得该锁时继续加一，释放锁时，计数器减一，当计数器值为0时，表明该锁未被任何线程所持有，其它线程可以竞争获取锁。

#### 什么是自旋

> 不要遇到 synchronized 就让等待锁的线程进入阻塞状态，而是让这个线程在 synchronized 边界做忙循环

很多 synchronized 里面的代码只是一些很简单的代码，执行时间非常快，既然 synchronized 里面的代码执行得非常快，不妨让等待锁的线程不要被阻塞，因为**线程阻塞涉及到用户态和内核态切换的问题开销很大**，而是**在 synchronized 的边界做忙循环**，这就是自旋。如果做了多次循环发现还没有获得锁，再阻塞，这样可能是一种更好的策略。

#### 多线程中 synchronized 锁升级的原理是什么？

> 目的：锁升级是为了减低了锁带来的性能消耗。

Java 的锁都是基于对象的，Java 对象有对象头，内容包括：

1. Mark Word，存储对象的 hashCode、锁信息等；
2. Class Metadata Address，存储到对象类型数据的指针；
3. 数组的长度（如果是数组）

每一个线程在准备获取共享资源时： 

第一步，检查锁的 MarkWord 里面是不是放的自己的 ThreadId，如果是，表示当前线程是处于 “**偏向锁**” ；

第二步，如果锁的 MarkWord 存放的不是自己的 ThreadId，这个时候会尝试使用 CAS 来替换 Mark Word 里面的线程 ID 为新线程的 ID，这个时候要分两种情况：

- CAS 替换成功，表示之前的线程不存在了， Mark Word 里面的线程 ID 为新线程的 ID，锁不会升级，仍然为偏向锁；
- CAS 替换失败，表示之前的线程仍然存在，根据锁的 MarkWord 里面的 ThreadId，通知该 ThreadId 的线程暂停，之前线程将 Markword 的内容置为空，升级为轻量级锁，会按照轻量级锁的方式进行竞争锁。

第三步，两个线程都把锁对象的 hashCode 复制到自己新建的用于存储锁的记录空间，接着开始通过 CAS 操作， 把锁对象的 MarkWord 的内容修改为自己新建的记录空间的地址的方式竞争 MarkWord；

第四步，第三步中成功执行 CAS 的获得资源，失败的则进入自旋 ；

第五步，自旋的线程在自旋过程中，如果成功获得资源（即之前获的资源的线程执行完成并释放了共享资源），则整个状态依然处于 **轻量级锁**的状态；如果自旋失败 （这边的自旋方式可以采用适应性自旋，简单来说就是线程如果自旋成功了，则下次自旋的次数会更多，如果自旋失败了，则自旋的次数就会减少）；

第六步，进入**重量级锁**的状态，这个时候，自旋的线程进入阻塞，等待竞争线程执行完成并唤醒自己。

#### 线程 B 怎么知道线程 A 修改了变量

* volatile 修饰变量

* synchronized 修饰修改变量的方法
* lock 对修改变量的代码块加锁

#### synchronized 和 Lock 有什么区别？

- synchronized 是 Java 关键字，而Lock 是个接口；
- synchronized 可以给静态方法、实例方法、代码块加锁，而 lock 只能给代码块加锁；
- synchronized 不需要手动获取锁和释放锁，而 lock 需要自己加锁和释放锁；
- 通过 Lock 可以知道有没有成功获取锁，而 synchronized 却无法办到。

#### synchronized 和 ReentrantLock 区别是什么？

**相同点：**

两者都是可重入锁

“可重入锁”概念是：自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，**如果不可锁重入的话，就会造成死锁**。同一个线程每次获取锁，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。

**不同点：**

* synchronized 是关键字，ReentrantLock 是类；
* ReentrantLock 只适用于代码块锁，而 synchronized 可以修饰类、方法、变量等；

- ReentrantLock 必须手动获取与释放锁，而 synchronized 不需要手动释放和开启锁；
- 二者的锁机制其实也是不一样的：
	ReentrantLock 底层调用的是 Unsafe 的park 方法加锁，synchronized 操作的应该是对象头中 mark word

### volatile

#### 为什么代码会重排序？

在执行程序时，为了**提高性能**，处理器和编译器常常会对指令进行重排序，但是不能随意重排序，不是你想怎么排序就怎么排序，它需要满足以下两个条件：

- **在单线程环境下不能改变程序运行的结果；**
- **存在数据依赖关系的不允许重排序**

需要注意的是：重排序不会影响单线程环境的执行结果，但是会破坏多线程的执行语义。

#### as-if-serial 规则和 happens-before 规则的区别

- as-if-serial 规则保证 **单线程** 内程序的执行结果不被改变，happens-before 规则保证 **正确同步的多线程** 程序的执行结果不被改变。
- as-if-serial 语义和 happens-before 这么做的目的，都是为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度。

#### volatile 关键字的作用

对于可见性，Java 提供了 volatile 关键字来保证可见性和有序性（禁止指令重排），但不能保证原子性。 volatile 提供 happens-before 的保证，确保一个线程的修改能对其他线程是可见的。当一个共享变量被 volatile 修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。

#### JMM内存屏障插入策略

**插入屏障**

StoreStore屏障 - volatile 写 - StoreLoad屏障

volatile读 - LoadLoad屏障 - LoadStore屏障

**volatile 与普通变量的重排序规则**

1. 如果第一个操作是 volatile 读，那无论第二个操作是什么，都不能重排序；
2. 如果第二个操作是 volatile 写，那无论第一个操作是什么，都不能重排序；
3. 如果第一个操作是 volatile 写，第二个操作是 volatile 读，那不能重排序。

#### volatile 能使得一个非原子操作变成原子操作吗？

volatile 只能保证可见性和有序性而不能保证原子性，但用 volatile 修饰 long 和 double 可以保证其操作原子性。

#### volatile 修饰符的有过什么实践？

单例模式-双重锁检验里面用 volatile 修饰实例变量。

#### volatile 和 synchronized 的区别是什么？

- volatile 是变量修饰符；synchronized 可以修饰类、方法、代码块；
- volatile 仅能实现变量的修改可见性和有序性，不能保证原子性，
	而 synchronized 则可以保证变量的修改原子性和可见性；
- volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞；
- **volatile关键字**是线程同步的**轻量级实现**，所以**volatile性能肯定比synchronized关键字要好**。synchronized关键字在JavaSE1.6之后进行了主要包括为了**减少获得锁和释放锁带来的性能消耗**而引入的偏向锁和轻量级锁以及其它各种优化之后执行效率有了显著提升，**实际开发中使用 synchronized 关键字的场景还是更多一些**。

## Lock 体系

### Lock 简介

#### Lock 接口是什么？对比同步它有什么优势？

Lock 接口比同步方法和同步块提供了更具扩展性的锁操作。

（1）可以使锁更**公平**

（2）可以使线程在**等待锁的时候响应中断**

（3）可以**让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间**

#### 乐观锁和悲观锁的理解及如何实现，有哪些实现方式？

悲观锁：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。
	传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。再比如 Java 里面的同步原语 synchronized 关键字的实现也是悲观锁。

乐观锁：顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。
	乐观锁适用于多读的应用类型，这样可以提高吞吐量，比如 atomic 包下面的原子变量类就是使用了乐观锁的一种实现方式 CAS 实现的。

#### 什么是 CAS

CAS 是 compare and swap 的缩写，即我们所说的比较交换，是一种乐观锁操作。

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值（B）。

如果内存地址 V 里面的值和 A 的值是一样的，那么就将内存里面的值更新成 B。

####  Java实现CAS的原理 - Unsafe类

在Java中，有一个`Unsafe`类，它在`sun.misc`包中。它里面是一些`native`方法（由底层的JVM使用C或者C++去实现），其中就有几个关于CAS的：

```java
boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);
boolean compareAndSwapInt(Object o, long offset,int expected,int x);
boolean compareAndSwapLong(Object o, long offset,long expected,long x);
```

当然，他们都是`public native`的。

Unsafe 中对 CAS 的实现是 C++ 写的，它的具体实现和操作系统、CPU 都有关系。

Linux 的 X86 下主要是通过`cmpxchgl`这个指令在 CPU 级完成 CAS 操作的，但在多处理器情况下必须使用`lock`指令加锁来完成。当然不同的操作系统和处理器的实现会有所不同，大家可以自行了解。

当然，Unsafe类里面还有其它方法用于不同的用途。比如支持线程挂起和恢复的`park`和`unpark`， LockSupport类底层就是调用了这两个方法。还有支持反射操作的`allocateInstance()`方法。

#### CAS 会产生什么问题？

1、**ABA 问题**：

比如说一个线程 one 从内存位置 V 中取出 A，这时候另一个线程 two 也从内存中取出 A，并且 two 进行了一些操作变成了 B，然后 two 又将 V 位置的数据变成 A，这时候线程 one 进行 CAS 操作发现内存中仍然是 A，然后 one 操作成功。尽管线程 one 的 CAS 操作成功，但可能存在潜藏的问题。可以在变量上加一个版本戳。从 Java1.5 开始 JDK 的 atomic 包里提供了一个类 AtomicStampedReference 来解决 ABA 问题。

2、**循环时间长开销大**：

对于资源竞争严重（线程冲突严重）的情况，CAS 自旋的概率会比较大，从而浪费更多的 CPU 资源，效率低于 synchronized。

3、**只能保证一个共享变量的原子操作**：

当对一个共享变量执行操作时，我们可以使用循环 CAS 的方式来保证原子操作，但是**对多个共享变量操作时，循环 CAS 就无法保证操作的原子性，这个时候就可以用锁**。

#### 死锁与活锁的区别，死锁与饥饿的区别？

> 区别：死锁动不了了；活锁一直重复尝试、失败、尝试、失败。

死锁：是指两个或两个以上的进程（或线程）在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。

活锁：任务或者执行者没有被阻塞，由于某些条件没有满足，导致**一直重复尝试，失败，尝试，失败**。

活锁和死锁的区别在于，处于活锁的实体是在不断的改变状态，这就是所谓的“活”， 而处于死锁的实体表现为等待；活锁有可能自行解开，死锁则不能。

饥饿：一个或者多个线程因为种种原因无法获得所需要的资源，导致一直无法执行的状态。

Java 中导致饥饿的原因：高优先级线程吞噬所有的低优先级线程的 CPU 时间。

### AQS详解

#### AQS 介绍

AQS 的全称为（AbstractQueuedSynchronizer），抽象队列同步器。

使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如 ReentrantReadWriteLock，SynchronousQueue，FutureTask 等等皆是基于 AQS 的。当然，我们自己也能利用 AQS 非常轻松容易地构造出符合我们自己需求的同步器。

#### AQS 原理分析

**如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。
如果被请求的共享资源被占用，那么就使用 CLH 队列，将暂时获取不到锁的线程加入到队列中。**

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 队列的一个结点（Node）来实现锁的分配。

看个AQS(AbstractQueuedSynchronizer)原理图：

![AQS原理图](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes11474088-c906e727d699fa9c.png)

AQS使用一个 volatile int 类型的成员变量 state 来表示同步状态，通过内置的 CLH 队列来完成获取资源线程的排队工作。AQS使用 CAS 对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state;	// 共享变量，使用volatile修饰保证线程可见性
```

状态信息通过 protected 类型的 getState，setState，compareAndSetState进行操作

```java
// 返回同步状态的当前值
protected final int getState() {  
    return state;
}
// 设置同步状态的值
protected final void setState(int newState) { 
    state = newState;
}
// 原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

**AQS 对资源的共享方式**

AQS定义两种资源共享方式

- **Exclusive**（独占）：只有一个线程能执行，如 ReentrantLock。又可分为公平锁和非公平锁：
	- 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
	- 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- **Share**（共享）：多个线程可同时执行，如 Semaphore/CountDownLatch。Semaphore、CountDownLatch、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。

**AQS底层使用了模板方法模式**

同步器的设计是基于模板方法模式的，如果需要自定义同步器（**模板方法模式**很经典的一个应用）：

1. 使用者继承 AQS 并重写指定的方法。（对共享资源 state 的获取和释放）
2. 调用 AQS 的模板方法，会进一步调用使用者重写的方法。

**AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：**

```java
isHeldExclusively()	// 该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)	// 独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)	// 独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)	// 共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)	// 共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

以 ReentrantLock 为例，state 初始化为0，表示未锁定状态。A 线程 lock() 时，会调用 tryAcquire() 独占该锁并将 state+1。此后，其他线程再 tryAcquire() 时就会失败，直到 A 线程 unlock( )到 state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。

以 CountDownLatch 以例，任务分为 N 个子线程去执行，state 也初始化为 N（注意N要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后 countDown() 一次，state 会 CAS 减 1。等到所有子线程都执行完后（即state=0），会 unpark() 主调用线程，然后主调用线程就会从 await() 函数返回，继续后续动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。

### 读写锁

首先明确一下，不是说 ReentrantLock 不好，只是 ReentrantLock 某些时候有局限。如果使用 ReentrantLock，可能本身是为了防止线程 A 在写数据、线程 B 在读数据造成的数据不一致，但这样，如果线程 C 在读数据、线程 D 也在读数据，读数据是不会改变数据的，没有必要加锁，但是还是加锁了，降低了程序的性能。因为这个，才诞生了读写锁 ReadWriteLock。

ReadWriteLock 是一个读写锁接口，读写锁是用来提升并发程序性能的锁分离技术，ReentrantReadWriteLock 是 ReadWriteLock 接口的一个具体实现，**实现了读写的分离，读锁是共享的，写锁是独占的**，读和读之间不会互斥，读和写、写和读、写和写之间才会互斥，提升了读写的性能。

## 并发容器

### ConcurrentHashMap

#### 什么是ConcurrentHashMap？

ConcurrentHashMap是Java中的一个**线程安全且高效的HashMap实现**。

那么它到底是如何实现线程安全的？

JDK 1.6版本关键要素：

- segment 继承了 ReentrantLock 充当锁的角色，为每一个 segment 提供了线程安全的保障；
- segment 维护了哈希散列表的若干个桶，每个桶由 HashEntry 构成的链表。

JDK1.8后，ConcurrentHashMap抛弃了原有的 Segment 分段锁，而**采用了 CAS + synchronized 来保证并发安全性**。

插入元素过程：

如果相应位置的Node还没有初始化，则调用CAS插入相应的数据；

如果相应位置的Node不为空，则对该节点加synchronized锁进行插入或更新操作。

### CopyOnWriteArrayList

CopyOnWrite容器即**写时复制的容器**，当我们往一个容器中添加元素的时候，不直接往容器中添加，而是将当前容器进行 copy，复制出来一个新的容器，然后向新容器中添加我们需要的元素，最后将原容器的引用指向新容器。

**CopyOnWriteArrayList 的使用场景**

通过源码分析，我们看出它的优缺点比较明显，所以使用场景也就比较明显。就是合适读多写少的场景。

**CopyOnWriteArrayList 的缺点**

1. 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致 young gc 或者 full gc。
2. **不能用于实时读的场景**，像拷贝数组、新增元素都需要时间，所以调用一个 set 操作后，**读取到数据可能还是旧的**，虽然 CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求。

### ThreadLocal

#### ThreadLocal的数据结构

<img src="https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesthreadlocal%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.a0bfadf8.png" alt="ThreadLocal数据结构" style="zoom: 67%;" />

每个线程 Thread 中有一个类型为 ThreadLocal.ThreadLocalMap 的实例变量 threadLocals，该实例变量中每个 entry为 <threadLocal的弱引用，value为强引用> 的映射，每个线程往 threadLocal 中对 value 进行操作时，都是在自己线程私有的 threadLocalMap 中进行操作，从而达到线程隔离。

```java
public class ThreadLocalDemo {
    // private static final ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>(){
    //     @Override
    //     protected Integer initialValue() {
    //         return 0;
    //     }
    // };

    private static final ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);

    public static void main(String[] args) {
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                for (int j = 0; j < 5; j++) {
                    Integer val = threadLocal.get();
                    threadLocal.set(++val);
                    System.out.println(Thread.currentThread().getName() + " ---- " + val);
                }
            }, "Thread-" + i).start();
        }

        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() + " ---- " + threadLocal.get());
    }
}
```

打印结果：启动了 2 个线程，每个线程最后都打印到 5， 最后threadLocal.get仍然是0

```java
Thread-1 ---- 1
Thread-0 ---- 1
Thread-1 ---- 2
Thread-0 ---- 2
Thread-1 ---- 3
Thread-1 ---- 4
Thread-1 ---- 5
Thread-0 ---- 3
Thread-0 ---- 4
Thread-0 ---- 5
main ---- 0
```

### ThreadLocal内存泄漏

#### ThreadLocal造成内存泄漏的原因？

ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用,而 value 是强引用。所以，如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，ThreadLocalMap 中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。ThreadLocalMap 实现中已经考虑了这种情况，每次使用完 ThreadLocal 后，都调用它的remove()方法，清理掉 key 为 null 的记录。

### BlockingQueue

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。

在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。

阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

JDK7 提供了 7 个阻塞队列。分别是：

> ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
> LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
> PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
> DelayQueue：一个使用优先级队列实现的无界阻塞队列。
> SynchronousQueue：一个不存储元素的阻塞队列。
> LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
> LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

Java 5 之前实现同步存取时，可以使用普通的一个集合，然后在使用线程的协作和线程同步可以实现生产者，消费者模式，主要的技术就是用好 wait，notify，notifyAll，synchronized 这些关键字。而在 Java 5 之后，可以使用阻塞队列来实现，此方式大大简少了代码量，使得多线程编程更加容易，安全方面也有保障。

BlockingQueue 接口是 Queue 的子接口，它的主要用途并不是作为容器，而是作为线程同步的的工具，因此他具有一个很明显的特性，**当生产者线程试图向 BlockingQueue 放入元素时，如果队列已满，则线程被阻塞，当消费者线程试图从中取出一个元素时，如果队列为空，则该线程会被阻塞**，正是因为它所具有这个特性，所以在程序中多个线程交替向 BlockingQueue 中放入元素，取出元素，它可以很好的控制线程之间的通信。

## 线程池

### Executors 创建四种常见线程池

#### 什么是线程池？有哪几种创建方式？

> 池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率。

在面向对象编程中，创建和销毁对象是很费时间的，因为创建一个对象要获取内存资源或者其它更多资源。在 Java 中更是如此，虚拟机将试图跟踪每一个对象，以便能够在对象销毁后进行垃圾回收。所以提高服务程序效率的一个手段就是**尽可能减少创建和销毁对象的次数**，特别是一些很耗资源的对象创建和销毁，这就是”池化资源”技术产生的原因。

线程池顾名思义就是事先创建若干个可执行的线程放入一个池（容器）中，需要的时候从池中获取线程不用自行创建，使用完毕不需要销毁线程而是放回池中，从而减少创建和销毁线程对象的开销。Java 5+中的 Executor 接口定义一个执行线程的工具。它的子类型即线程池接口是 ExecutorService。要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，因此在工具类 Executors 面提供了一些静态工厂方法，生成一些常用的线程池，如下所示：

（1）Executors.newSingleThreadExecutor：`1, 1, new LinkedBlockingQueue`。所有任务按照**先来先执行**的顺序执行。如果这个唯一的线程不空闲，那么新来的任务会存储在任务队列里等待执行。（由于阻塞队列默认大小为Integer.MAX_VALUE，因此可能OOM）

（2）Executors.newFixedThreadPool：`nCoreThreads, nCoreThreads, new LinkedBlockingQueue`，所以只能创建核心线程，不能创建非核心线程。因为 LinkedBlockingQueue 的默认大小是 Integer.MAX_VALUE，故如果核心线程空闲，则交给核心线程处理；如果核心线程不空闲，则入列等待，直到核心线程空闲。（由于阻塞队列默认大小为Integer.MAX_VALUE，因此可能OOM）

（3） Executors.newCachedThreadPool：`0, Integer.MAX_VALUE`，不创建核心线程，线程池最大为Integer.MAX_VALUE。（线程池太大导致OOM）

newCachedThreadPool 和 newFixedThreadPool 都几乎不会触发拒绝策略，但是原理不同。
FixedThreadPool 是因为阻塞队列可以很大（最大为Integer最大值），故几乎不会触发拒绝策略；CachedThreadPool 是因为线程池很大（最大为Integer最大值），几乎不会导致线程数量大于最大线程数，故几乎不会触发拒绝策略。

（4）Executors.newScheduledThreadPool：`nCoreThreads, Integer.MAX_VALUE`创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。

#### 线程池有什么优点？

- 降低资源消耗：重用存在的线程，减少对象创建销毁的开销；
- 提高响应速度：当任务到达时，任务可以不需要的等到线程创建就能立即执行；
- 提高线程的可管理性：使用线程池可以进行统一的分配，调优和监控。

#### 线程池都有哪些状态？

- RUNNING：接受新的任务提交，处理等待队列中的任务；
- SHUTDOWN：不接受新的任务提交，但是会继续处理等待队列中的任务；
- STOP：不接受新的任务提交，不再处理等待队列中的任务，中断正在执行任务的线程；
- TIDYING：所有的任务都销毁了，workCount 为 0，线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()；
- TERMINATED：terminated()方法结束后，线程池的状态就会变成这个。

#### 在 Java 中 Executor 和 Executors 的区别？

- **Executors 工具类**的不同方法按照我们的需求**创建了不同的线程池**，来满足业务的需求；
- **Executor 接口对象执行我们的线程任务**，ExecutorService 接口继承了 Executor 接口并进行了扩展，提供了更多的方法我们能获得任务执行的状态并且可以获取任务的返回值。

#### 线程池中 submit() 和 execute() 方法有什么区别？

接收参数：submit() 可以执行 Runnable 和 Callable 类型的任务，而execute()只能执行 Runnable 类型的任务；

返回值：submit() 方法可以返回持有计算结果的 Future 对象，而 execute() 没有；

异常处理：submit() 方便 Exception 处理。

### ThreadPoolExecutor 自定义线程池

#### Executors

《阿里巴巴Java开发手册》中强制线程池不允许使用 Executors 去创建，而是**通过 ThreadPoolExecutor 的方式**，这样的处理方式让写的同学**更加明确线程池的运行规则，规避资源耗尽的风险**

Executors 各个方法的弊端：

- newSingleThreadExecutor 和 newFixedThreadPool :
	`0, 0`和 `n, n` ，但是使用 LinkedBlockingQueue，最大可以为 Integer.MAX_VALUE

	主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至 OOM。

- newCachedThreadPool 和 newScheduledThreadPool:
	`0, Integer.MAX_VALUE` 和 `n, Integer.MAX_VALUE`

	主要问题是线程数最大数是 Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至 OOM

ThreaPoolExecutor创建线程池方式只有一种，就是走它的构造函数，参数自己指定

#### ThreaPoolExecutor 

创建线程池的方式有多种，这里你只需要答 ThreadPoolExecutor 即可。

ThreadPoolExecutor() 是最原始的线程池创建，也是阿里巴巴 Java 开发手册中明确规范的创建线程池的方式。

#### ThreadPoolExecutor构造函数重要参数分析

**`ThreadPoolExecutor`** **3 个最重要的参数：**

- **`corePoolSize`** ：核心线程数，线程数定义了最小可以同时运行的线程数量
- **`maximumPoolSize`** ：线程池中允许存在的工作线程的最大数量
- **`workQueue`**：当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，任务就会被存放在等待队列中

`ThreadPoolExecutor`其他常见参数:

1. **`keepAliveTime`**：非核心线程如果处于闲置状态超过该值，就会被销毁。
2. **`unit`** ：`keepAliveTime` 闲置销毁时长的时间单位
3. **`threadFactory`**：为线程池提供创建新线程的线程工厂
4. **`handler`** ：线程池任务队列超过 maxinumPoolSize 之后的拒绝策略

#### ThreadPoolExecutor拒绝策略

**`ThreadPoolExecutor`** **拒绝策略定义:**

如果当前同时运行的线程数量达到最大线程数量并且等待队列也已经被放满时，`ThreadPoolTaskExecutor` 定义一些策略:

- **`ThreadPoolExecutor.AbortPolicy`**：抛出 `RejectedExecutionException`来拒绝新任务的处理。
- **`ThreadPoolExecutor.CallerRunsPolicy`**：调用执行自己的线程运行任务。
- **`ThreadPoolExecutor.DiscardPolicy`**：不处理新任务，直接丢弃掉。
- **`ThreadPoolExecutor.DiscardOldestPolicy`**： 此策略将丢弃最早的未处理的任务请求。

#### 一个简单的线程池Demo:`Runnable` + `ThreadPoolExecutor`

线程池实现原理

![线程池实现原理](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesaHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMS8yNS8xNmVhMDQ3NjEyOTVlNzY2.png)

**总结一下处理流程**

1. 线程总数量 < corePoolSize，无论线程是否空闲，都会新建一个核心线程执行任务（让核心线程数量快速达到corePoolSize，在核心线程数量 < corePoolSize时）。**注意，这一步需要获得全局锁。**
2. 线程总数量 >= corePoolSize时，新来的线程任务会进入等待队列中等待，然后空闲的核心线程会依次去缓存队列中取任务来执行（体现了**线程复用**）。 
3. 当等待队列满了，说明这个时候任务已经多到爆棚，需要一些“临时工”来执行这些任务了。于是会创建非核心线程去执行这个任务。**注意，这一步需要获得全局锁。**
4. 缓存队列之前满了， 现在加非核心线程且总线程数达到了maximumPoolSize，则会采取上面提到的拒绝策略进行处理。

整个过程如图所示：

为了让大家更清楚上面的面试题中的一些概念，我写了一个简单的线程池 Demo。

首先创建一个 `Runnable` 接口的实现类（当然也可以是 `Callable` 接口，我们上面也说了两者的区别。）

```java
import Java.util.Date;

/**
 * 这是一个简单的Runnable类，需要大约5秒钟来执行其任务。
 */
public class MyRunnable implements Runnable {

    private String command;

    public MyRunnable(String s) {
        this.command = s;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " Start. Time = " + new Date());
        processCommand();
        System.out.println(Thread.currentThread().getName() + " End. Time = " + new Date());
    }

    private void processCommand() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public String toString() {
        return this.command;
    }
}
```

编写测试程序，我们这里以阿里巴巴推荐的使用 `ThreadPoolExecutor` 构造函数自定义参数的方式来创建线程池。

```java
import Java.util.concurrent.ArrayBlockingQueue;
import Java.util.concurrent.ThreadPoolExecutor;
import Java.util.concurrent.TimeUnit;

public class ThreadPoolExecutorDemo {

    private static final int CORE_POOL_SIZE = 5;
    private static final int MAX_POOL_SIZE = 10;
    private static final int QUEUE_CAPACITY = 100;
    private static final Long KEEP_ALIVE_TIME = 1L;
    public static void main(String[] args) {

        //使用阿里巴巴推荐的创建线程池的方式
        //通过ThreadPoolExecutor构造函数自定义参数创建
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                CORE_POOL_SIZE,
                MAX_POOL_SIZE,
                KEEP_ALIVE_TIME,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(QUEUE_CAPACITY),
                new ThreadPoolExecutor.CallerRunsPolicy());

        for (int i = 0; i < 10; i++) {
            //创建WorkerThread对象（WorkerThread类实现了Runnable 接口）
            Runnable worker = new MyRunnable("" + i);
            //执行Runnable
            executor.execute(worker);
        }
        //终止线程池
        executor.shutdown();
        while (!executor.isTerminated()) {
        }
        System.out.println("Finished all threads");
    }
}
```

可以看到我们上面的代码指定了：

1. `corePoolSize`: 核心线程数为 5。
2. `maximumPoolSize` ：最大线程数 10
3. `keepAliveTime` : 等待时间为 1L。
4. `unit`: 等待时间的单位为 TimeUnit.SECONDS。
5. `workQueue`：任务队列为 `ArrayBlockingQueue`，并且容量为 100;
6. `handler`:饱和策略为 `CallerRunsPolicy`。

**Output：**

```java
pool-1-thread-2 Start. Time = Tue Nov 12 20:59:44 CST 2019
pool-1-thread-5 Start. Time = Tue Nov 12 20:59:44 CST 2019
pool-1-thread-4 Start. Time = Tue Nov 12 20:59:44 CST 2019
pool-1-thread-1 Start. Time = Tue Nov 12 20:59:44 CST 2019
pool-1-thread-3 Start. Time = Tue Nov 12 20:59:44 CST 2019
pool-1-thread-5 End. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-3 End. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-2 End. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-4 End. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-1 End. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-2 Start. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-1 Start. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-4 Start. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-3 Start. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-5 Start. Time = Tue Nov 12 20:59:49 CST 2019
pool-1-thread-2 End. Time = Tue Nov 12 20:59:54 CST 2019
pool-1-thread-3 End. Time = Tue Nov 12 20:59:54 CST 2019
pool-1-thread-4 End. Time = Tue Nov 12 20:59:54 CST 2019
pool-1-thread-5 End. Time = Tue Nov 12 20:59:54 CST 2019
pool-1-thread-1 End. Time = Tue Nov 12 20:59:54 CST 2019
```

## 并发工具

### CountDownLatch 与 CyclicBarrier

CountDownLatch 与 CyclicBarrier 都是用于控制并发的工具类，都可以理解成维护的就是一个计数器，但是这两者还是各有不同侧重点的：

- CountDownLatch 强调一个线程等多个线程完成某件事情。CyclicBarrier 是多个线程互等，等大家都完成，再携手共进；
- 调用 CountDownLatch 的 countDown 方法后，当前线程并不会阻塞，会继续往下执行；而调用CyclicBarrier 的 await 方法，会阻塞当前线程，直到 CyclicBarrier 指定的线程全部都到达了指定点的时候，才能继续往下执行；
- CountDownLatch 是不能复用的，而 CyclicBarrier 是可以复用的。

### Semaphore

**Semaphore(信号量)-允许多个线程同时访问：** synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。

Semaphore 就是一个信号量，它的作用是限制某段代码块的并发数。Semaphore有一个构造函数，可以传入一个 int 型整数 n，表示某段代码最多只有 n 个线程可以访问，如果超出了 n，那么请等待，等到某个线程执行完毕这段代码块，下一个线程再进入。由此可以看出如果 Semaphore 构造函数中传入的 int 型整数 n=1，相当于变成了一个 synchronized 了。

## 模拟

### 死锁

```java
public class DeadLock {
    private static Object A = new Object();
    private static Object B = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (A) {
                System.out.println(Thread.currentThread().getName() + " 已获得资源A");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " 等待获得资源B");
                synchronized (B) {
                    System.out.println(Thread.currentThread().getName() + " 已获得资源B");
                }
            }
        }, "线程1").start();

        new Thread(() -> {
            synchronized (B) {
                System.out.println(Thread.currentThread().getName() + " 已获得资源B");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " 等待获得资源A");
                synchronized (A) {
                    System.out.println(Thread.currentThread().getName() + " 已获得资源A");
                }
            }
        }, "线程2").start();
    }
}
```

> 线程1 已获得资源A
> 线程2 已获得资源B
> 线程1 等待获得资源B
> 线程2 等待获得资源A

### run方法和start方法

```java
public class DiffStartRun {
    public static void main(String[] args) {
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName());
        }, "线程A").run();

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName());
        }, "线程B").start();
    }
}
```

> main
> 线程B
>
> new 一个 Thread，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，第一个所谓的线程A只是 main 线程的一个方法，也就是说其所在线程还是 main 线程，所以这并不是多线程工作。
>
> **总结： 调用 start() 方法方可启动线程并使线程进入就绪状态，直接执行 run() 方法的话不会以多线程的方式执行。**

### 两个线程，交替加减

1. synchronized

	```java
	/**
	 * @author fangyaxing
	 * @date 2022/9/12
	 */
	public class TwoThreadWithSync {
	    private static int value = 0;
	
	    private static final Object lock = new Object();
	
	    public static void main(String[] args) {
	        new Thread(() -> {
	            for (int i = 0; i < 5; i++) {
	                synchronized (lock) {
	                    while (value != 0) {
	                        try {
	                            lock.wait();
	                        } catch (InterruptedException e) {
	                            e.printStackTrace();
	                        }
	                    }
	                    value++;
	                    System.out.println(Thread.currentThread().getName() + " ---- " + value);
	                    lock.notifyAll();
	                }
	            }
	        }, "Thread-A").start();
	
	        new Thread(() -> {
	            for (int i = 0; i < 5; i++) {
	                synchronized (lock) {
	                    while (value != 1) {
	                        try {
	                            lock.wait();
	                        } catch (InterruptedException e) {
	                            e.printStackTrace();
	                        }
	                    }
	                    value--;
	                    System.out.println(Thread.currentThread().getName() + " ---- " + value);
	                    lock.notifyAll();
	                }
	            }
	        }, "Thread-B").start();
	    }
	
	}
	```

2. lock + condition

	```java
	/**
	 * @author fangyaxing
	 * @date 2022/9/12
	 */
	public class TwoThreadWithLock {
	    private static int value = 0;
	
	    private static final Lock lock = new ReentrantLock();
	
	    static Condition a = lock.newCondition();
	
	    static Condition b = lock.newCondition();
	
	    public static void main(String[] args) {
	        new Thread(() -> {
	            for (int i = 0; i < 5; i++) {
	                try {
	                    lock.lock();
	                    while (value != 0) {
	                        a.await();
	                    }
	                    value++;
	                    System.out.println(Thread.currentThread().getName() + " ---- " + value);
	                    b.signal();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                } finally {
	                    lock.unlock();
	                }
	            }
	        }, "Thread-A").start();
	
	        new Thread(() -> {
	            for (int i = 0; i < 5; i++) {
	                try {
	                    lock.lock();
	                    while (value != 1) {
	                        b.await();
	                    }
	                    value--;
	                    System.out.println(Thread.currentThread().getName() + " ---- " + value);
	                    a.signal();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                } finally {
	                    lock.unlock();
	                }
	            }
	        }, "Thread-B").start();
	    }
	}
	```

### 三个线程，循环打印

```java
public class Demo5 {
    private int value = 0;
    private Lock lock = new ReentrantLock();
    private Condition a = lock.newCondition();
    private Condition b = lock.newCondition();
    private Condition c = lock.newCondition();

    public void outputA(int round) {
        try {
            lock.lock();
            while (value != 0) {
                a.await();
            }
            value = 1;
            System.out.println("第" + round + "轮，" + Thread.currentThread().getName() + "输出A");
            b.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void outputB(int round) {
        try {
            lock.lock();
            while (value != 1) {
                b.await();
            }
            value = 2;
            System.out.println("第" + round + "轮，" + Thread.currentThread().getName() + "输出B");
            c.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void outputC(int round) {
        try {
            lock.lock();
            while (value != 2) {
                c.await();
            }
            value = 0;
            System.out.println("第" + round + "轮，" + Thread.currentThread().getName() + "输出C");
            a.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        Demo5 s = new Demo5();

        new Thread(() -> {
            for (int i = 0; i < 3; i++) {
                s.outputA(i);
            }
        }, "Thread-A").start();

        new Thread(() -> {
            for (int i = 0; i < 3; i++) {
                s.outputB(i);
            }
        }, "Thread-B").start();

        new Thread(() -> {
            for (int i = 0; i < 3; i++) {
                s.outputC(i);
            }
        }, "Thread-C").start();

    }
}
```
