---
layout:     post
title:      Synchronized和ReentrantLock
subtitle:   概述
date:       2021-05-28
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - 多线程
---

# synchronized和ReentrantLock

## synchronized

### 类锁和对象锁

​	synchronized关键字常用于锁方法、锁代码块和锁对象

~~~java
class A{
	public void synchronized f1(){}
    public static void synchronized f2(){}
}
~~~

​	上面使用synchronized等价于下面代码

~~~java
class A{
	public void f1(){
        synchronized(this){
            ...
        }
    }
    public static void f2(){
        synchronized(A.class){
            ...
        }
    }
}
~~~

​	**所以对于非静态函数，锁其实是加在对象上面的。对于静态成员函数锁是加在类上的。**一个是对象锁一个是类锁。

​	所以如果一个静态函数和一个非静态函数都加了synchronized，分别被两个线程调用，不会互斥，因为是两把不同的锁。

### synchronized关键字的原理

#### 对象头中的MarkWord

​	每个对象都有一个对象头叫MarkWord（标记字段）

​	Mark Word记录了对象和锁有关的信息，默认存储对象的**hashcode、GC分代年龄、和锁标志位信息**。当这个对象被synchronized关键字当成同步锁时，围绕这个锁的一系列操作都和Mark Word有关。

![image-20210528214722693](C:\Users\ql\AppData\Roaming\Typora\typora-user-images\image-20210528214722693.png)

![image-20210528225430191](C:\Users\ql\AppData\Roaming\Typora\typora-user-images\image-20210528225430191.png)

#### monitor（监视器锁）

​	**JVM基于进入和退出Monitor对象来实现方法同步和代码块同步。**代码块同步是使用monitorenter和monitorexit指令实现的，**monitorenter指令是在编译后插入到同步代码块的开始位置**，**而monitorexit是插入到方法结束处和异常处。**任何对象都有一个monitor与之关联，**当且一个monitor被持有后，它将处于锁定状态。**

​	根据虚拟机规范的要求，在执行monitorenter指令时，首先要去尝试获取对象的锁，如果这个对象没被锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加1；相应地，在执行monitorexit指令时会将锁计数器减1，当计数器被减到0时，锁就释放了。如果获取对象锁失败了，那当前线程就要阻塞等待，直到对象锁被另一个线程释放为止。

​	反编译可以发现，对象头，他会关联到一个monitor对象。**Monitor是依赖于底层的操作系统的Mutex Lock（互斥锁）来实现的线程同步。**

- 当我们进入一个人方法的时候，执行**monitorenter**，就会获取当前对象的一个所有权，这个时候monitor进入数为1，当前的这个线程就是这个monitor的owner。
- 如果你已经是这个monitor的owner了，你再次进入，就会把进入数+1.
- 同理，当他执行完**monitorexit**，对应的进入数就-1，直到为0，才可以被其他线程持有。

### 锁升级过程

​	**Synchronized是通过对象内部的一个叫做监视器锁（monitor）来实现的，监视器锁本质又是依赖于底层的操作系统的Mutex Lock（互斥锁）来实现的。**而操作系统实现线程之间的切换需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么Synchronized效率低的原因。**因此，这种依赖于操作系统Mutex Lock所实现的锁我们称之为“重量级锁”。**

Java SE 1.6为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”：锁一共有4种状态，级别从低到高依次是：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态。锁可以升级但不能降级。

​	**偏向锁通过对比Mark Word解决加锁问题，避免执行CAS操作。而轻量级锁是通过用CAS操作和自旋来解决加锁问题，避免线程阻塞和唤醒而影响性能。重量级锁是将除了拥有锁的线程以外的线程都阻塞。**

![image-20210528221120625](C:\Users\ql\AppData\Roaming\Typora\typora-user-images\image-20210528221120625.png)

#### 无锁

​	无锁没有对资源进行锁定，所有线程都可以访问并修改一个资源，但同时只有一个线程可以修改成功。CAS原理及应用即是无锁的实现。

#### 偏向锁

​	偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁，**降低获取锁的代价**。

​	在大多数情况下，锁总是由同一线程多次获得，不存在多线程竞争，所以出现了偏向锁。其目标就是在只有一个线程执行同步代码块时能够提高性能。

​	当一个线程访问同步代码块并获取锁时，**会在Mark Word里存储锁偏向的线程ID**。在线程进入和退出同步块时不再通过CAS操作来加锁和解锁，而是检测Mark Word里是否存储着指向当前线程的偏向锁。

#### 轻量级锁

​	是指当锁是偏向锁的时候，被另外的线程所访问，偏向锁就会升级为轻量级锁，**其他线程会通过自旋CAS的形式尝试获取锁，不会阻塞，从而提高性能。**

​	若当前只有一个等待线程，则该线程通过自旋进行等待。**但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁升级为重量级锁。**

#### 重量级锁

​	升级为重量级锁时，锁标志的状态值变为“10”，此时Mark Word中存储的是指向重量级锁的指针，此时等待锁的线程都会进入阻塞状态。

## ReentrantLock

​	

## synchronizd和ReentrantLock异同

**不同点**

- 一个线程获取锁之后，另一个线程处于阻塞状态，被synchronized阻塞的是不可中断的，而Lock的tryLock方法是可以被中断的。
- ReentrantLock的lock.tryLock(long timeout,TimeUnit unit)方法可以实现限时锁，参数为时间和单位
- ReentrantLock可以手动选择公平锁还是非公平锁

**相同点**

- 性能基本持平，ReentrantLock只是提供了synchronized更丰富的功能