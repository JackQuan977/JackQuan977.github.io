---
layout:     post
title:      BIO,NIO,AIO复习
subtitle:   概述
date:       2021-02-22
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - JavaSE
---

# BIO,NIO,AIO复习

​	BIO,NIO,AIO可以理解为java语言对操作系统各种IO模型的封装

### BIO（Blocking I/O)

- 同步阻塞IO模式，数据的读取和写入必须阻塞在一个线程内等待其，完成。**一请求一应答通信模型**。

- 如果要处理多个客户端请求，就必须使用多线程。线程创建销毁成本很高，可以用线程池降低创建和回收成本。

- 在活动连接数不是很高（小于单机1000）情况下，这种模型是不错的。

### NIO(New I/O)

- 同步非阻塞模式。支持面向缓冲，基于通道的I/O操作方法
- I/O面向流，NIO面向缓冲区，NIO直接读到缓冲区中进行操作
- NIO通过channel通道进行读写，通道是双向的可读可写，而流的读写是单向的。无论读写只能和Buffer交互
- 但是JDK原生NIO编程复杂，还会出现很多bug，Netty的出现改善了原生NIO存在的问题

### AIO（Asynchronous I/O)

- AIO也就是NIO2，是异步非阻塞

