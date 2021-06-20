---
layout:     post
title:      Semaphore的使用及源码分析
subtitle:   概述
date:       2021-06-17
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - 多线程
---

# Semaphore的使用及源码分析

### Semaphore 是什么？

​	Semaphore 一般译作 `信号量`，它也是一种线程同步工具，主要用于多个线程对共享资源进行并行操作的一种工具类。它代表了一种`许可`的概念，是否允许多线程对同一资源进行操作的许可，**使用 Semaphore 可以控制并发访问资源的线程个数。**

### Semaphore 的使用场景

​	Semaphore 的使用场景主要用于`流量控制`，比如数据库连接，同时使用的数据库连接会有数量限制，数据库连接不能超过一定的数量，当连接到达了限制数量后，后面的线程只能排队等前面的线程释放数据库连接后才能获得数据库连接。

​	再比如交通公路上的红绿灯，绿灯亮起时只能让 100 辆车通过，红灯亮起不允许车辆通过。

再比如停车场的场景中，一个停车场有有限数量的车位，同时能够容纳多少台车，车位满了之后只有等里面的车离开停车场外面的车才可以进入。

### Semaphore用法

#### 构造方法

~~~java
public Semaphore(int permits)
public Semaphore(int permits, boolean fair)
~~~

- permits 表示许可线程的数量
- fair 表示公平性，如果这个设为 true 的话，下次执行的线程会是等待最久的线程

#### 重要方法

~~~java
acquire()  
获取一个令牌，在获取到令牌、或者被其他线程调用中断之前线程一直处于阻塞状态。

acquire(int permits)  
获取一个令牌，在获取到令牌、或者被其他线程调用中断、或超时之前线程一直处于阻塞状态。
    
acquireUninterruptibly() 
获取一个令牌，在获取到令牌之前线程一直处于阻塞状态（忽略中断）。
    
tryAcquire()
尝试获得令牌，返回获取令牌成功或失败，不阻塞线程。

tryAcquire(long timeout, TimeUnit unit)
尝试获得令牌，在超时时间内循环尝试获取，直到尝试获取成功或超时返回，不阻塞线程。

release()
释放一个令牌，唤醒一个获取令牌不成功的阻塞线程。

hasQueuedThreads()
等待队列里是否还存在等待线程。

getQueueLength()
获取等待队列里阻塞的线程数。

drainPermits()
清空令牌把可用令牌数置为0，返回清空令牌的数量。

availablePermits()
返回可用的令牌数量。
~~~

### Semaphore 使用

​	下面我们就来模拟一下停车场的业务场景：在进入停车场之前会有一个提示牌，上面显示着停车位还有多少，当车位为 0 时，不能进入停车场，当车位不为 0 时，才会允许车辆进入停车场。所以停车场有几个关键因素：停车场车位的总容量，当一辆车进入时，停车场车位的总容量 - 1，当一辆车离开时，总容量 + 1，停车场车位不足时，车辆只能在停车场外等待。

~~~java
public class CarParking {

    private static Semaphore semaphore = new Semaphore(10);

    public static void main(String[] args){

        for(int i = 0;i< 100;i++){

            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println("欢迎 " + Thread.currentThread().getName() + " 来到停车场");
                    // 判断是否允许停车
                    if(semaphore.availablePermits() == 0) {
                        System.out.println("车位不足，请耐心等待");
                    }
                    try {
                        // 尝试获取
                        semaphore.acquire();
                        System.out.println(Thread.currentThread().getName() + " 进入停车场");
                        Thread.sleep(new Random().nextInt(10000));// 模拟车辆在停车场停留的时间
                        System.out.println(Thread.currentThread().getName() + " 驶出停车场");
                        semaphore.release();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, i + "号车");

            thread.start();
        }

    }

}
~~~

在上面这段代码中，我们给出了 Semaphore 的初始容量，也就是只有 10 个车位，我们用这 10 个车位来控制 100 辆车的流量，所以结果和我们预想的很相似，即大部分车都在等待状态。但是同时仍允许一些车驶入停车场，驶入停车场的车辆，就会 semaphore.acquire 占用一个车位，驶出停车场时，就会 semaphore.release 让出一个车位，让后面的车再次驶入。

### Semaphore 信号量的模型

​	上面代码虽然比较简单，但是却能让我们了解到一个信号量模型的`五脏六腑`。下面是一个信号量的模型：

![image-20210617134332655](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210617134332655.png)

​	来解释一下 Semaphore ，Semaphore 有一个初始容量，这个初始容量就是 Semaphore 所能够允许的信号量。在调用 Semaphore 中的 acquire 方法后，Semaphore 的容量 -1，相对的在调用 release 方法后，Semaphore 的容量 + 1，在这个过程中，计数器一直在监控 Semaphore 数量的变化，等到流量超过 Semaphore 的容量后，多余的流量就会放入等待队列中进行排队等待。等到 Semaphore 的容量允许后，方可重新进入。

> Semaphore 所控制的流量其实就是一个个的线程，因为并发工具最主要的研究对象就是线程。

它的工作流程如下

![image-20210617135133556](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210617135133556.png)

### Semaphore 深入理解

#### Semaphore 基本属性

​	Semaphore 中只有一个属性

~~~java
private final Sync sync;
~~~

​	**Sync 是 Semaphore 的同步实现**，Semaphore 保证线程安全性的方式和 ReentrantLock 、CountDownLatch 类似，都是继承于 AQS 的实现。同样的，这个 Sync 也是继承于 `AbstractQueuedSynchronizer` 的一个变量，也就是说，聊 Semaphore 也绕不开 AQS，所以说 AQS 真的太重要了。

#### Semaphore 的公平性和非公平性

那么我们进入 Sync 内部看看它实现了哪些方法

~~~java
abstract static class Sync extends AbstractQueuedSynchronizer {
  private static final long serialVersionUID = 1192457210091910933L;

  Sync(int permits) {
    setState(permits);
  }

  final int getPermits() {
    return getState();
  }

  final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
      int available = getState();
      int remaining = available - acquires;
      if (remaining < 0 ||
          compareAndSetState(available, remaining))
        return remaining;
    }
  }

  protected final boolean tryReleaseShared(int releases) {
    for (;;) {
      int current = getState();
      int next = current + releases;
      if (next < current) // overflow
        throw new Error("Maximum permit count exceeded");
      if (compareAndSetState(current, next))
        return true;
    }
  }

  final void reducePermits(int reductions) {
    for (;;) {
      int current = getState();
      int next = current - reductions;
      if (next > current) // underflow
        throw new Error("Permit count underflow");
      if (compareAndSetState(current, next))
        return;
    }
  }

  final int drainPermits() {
    for (;;) {
      int current = getState();
      if (current == 0 || compareAndSetState(current, 0))
        return current;
    }
  }
}
~~~

​	首先是 Sync 的初始化，内部调用了 `setState` 并传递了 permits ，我们知道，AQS 中的 State 其实就是同步状态的值，而 Semaphore 的这个 permits 就是代表了许可的数量。

​	getPermits 其实就是调用了 getState 方法获取了一下线程同步状态值。后面的 nonfairTryAcquireShared 方法其实是在 Semaphore 中构造了 NonfairSync 中的 tryAcquireShared 调用的

​	事实上，Semaphore 就像 ReentrantLock 一样，也存在“公平”和"不公平"两种，默认情况下 Semaphore 是一种不公平的信号量

​	Semaphore 的不公平意味着它不会保证线程获得许可的顺序，Semaphore 会在线程等待之前为调用 acquire 的线程分配一个许可，拥有这个许可的线程会自动将自己置于线程等待队列的头部。

当这个参数为 true 时，Semaphore 确保任何调用 acquire 的方法，都会按照先入先出的顺序来获取许可。

~~~java
final int nonfairTryAcquireShared(int acquires) {
  for (;;) {
    // 获取同步状态值
    int available = getState();
    // state 的值 - 当前线程需要获取的信号量（通常默认是 -1），只有
    // remaining > 0 才表示可以获取。
    int remaining = available - acquires;
    // 先判断是否小于 0 ，如果小于 0 则表示无法获取，如果是正数
    // 就需要使用 CAS 判断内存值和同步状态值是否一致，然后更新为同步状态值 - 1
    if (remaining < 0 ||
        compareAndSetState(available, remaining))
      return remaining;
  }
}
~~~

