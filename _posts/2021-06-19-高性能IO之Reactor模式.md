---
layout:     post
title:      高性能IO之Reactor模式
subtitle:   概述
date:       2021-06-19
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - Netty
---

# 高性能IO之Reactor模式

### **什么是Reactor模式**

​	Reactor模式也叫反应器模式，大多数IO相关组件如Netty、Redis在使用的IO模式

​	Reactor模式是一种设计模式，它是基于事件驱动的，可以并发的处理多个服务请求，当请求抵达后，依据多路复用策略，同步的派发这些请求至相关的请求处理程序。

### 为什么使用Reactor模式

####  多线程IO的不足

​	最最原始的网络编程思路就是服务器用一个while循环，不断监听端口是否有新的套接字连接，如果有，那么就调用一个处理函数处理，类似：

~~~java
while(true){

socket = accept();

handle(socket)

}
~~~

​	这种方法的最大问题是无法并发，效率太低，如果当前的请求没有处理完，那么后面的请求只能被阻塞，服务器的吞吐量太低。之后，想到了使用多线程，每一个连接用一个线程处理。

​	缺点在于资源要求太高，系统中创建线程是需要比较高的系统资源的，如果连接数太高，系统无法承受，而且，线程的反复创建-销毁也需要代价。一个线程如果没有数据可读会一直阻塞在那占用线程资源。线程池本身可以缓解线程创建-销毁的代价，这样优化确实会好很多，但还是无法从根本上上解决问题。

### **Reactor模型**

​	**在Reactor中，这些被拆分的小线程或者子过程对应的是handler，每一种handler会出处理一种event。这里会有一个全局的管理者selector，我们需要把channel注册感兴趣的事件，那么这个selector就会不断在channel上检测是否有该类型的事件发生，如果没有，那么主线程就会被阻塞，否则就会调用相应的事件处理函数即handler来处理。**

#### Reactor模式的的演化过程如下：

**采用基于事件驱动的设计，当有事件触发时，才会调用处理器进行数据处理。**

![image-20210619140546943](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210619140546943.png)

Reactor：负责响应IO事件，当检测到一个新的事件，将其发送给相应的Handler去处理。

Handler：负责处理非阻塞的行为，标识系统管理的资源；同时将handler与事件绑定。

Reactor为单个线程，需要处理accept连接，同时发送请求到处理器中。

由于只有单个线程，所以处理器中的业务需要能够快速处理完。

**使用多线程处理业务逻辑。**

![image-20210619140731162](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210619140731162.png)

将处理器的执行放入线程池，多线程进行业务处理。但Reactor仍为单个线程。

**对于多个CPU的机器，为充分利用系统资源，将Reactor拆分为两部分。**

![image-20210619140838505](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210619140838505.png)

mainReactor负责监听连接，accept连接给subReactor处理，为什么要单独分一个Reactor来处理监听呢？因为像TCP这样需要经过3次握手才能建立连接，这个建立连接的过程也是要耗时间和资源的，单独分一个Reactor来处理，可以提高性能。