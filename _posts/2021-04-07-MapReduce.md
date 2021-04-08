---
layout:     post
title:      MapReduce
subtitle:   概述
date:       2021-04-07
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - 大数据
---

# MapReduce

​	MapReduce 既是一个编程模型，又是一个计算框架。也就是说，开发人员必须基于 MapReduce 编程模型进行编程开发，然后将程序通过 MapReduce 计算框架分发到 Hadoop 集群中运行。MapReduce充分借鉴了分而治之的思想，将一个数据的处理过程分为Map（映射）和Reduce（处理）两步。那么用户只需要将数据以需要的格式交给reduce函数处理就能轻松实现分布式的计算，很多的工作都由mapReduce框架为我们封装好，大大简化了操作流程。

### MapReduce的编程思想

​	MapReduce数据操作的**最小单位是一个键值对**。用户在使用MapReduce编程模型的时候第一步就是要将数据转化为键值对的形式，map函数会以键值对作为输入，**经过map函数的处理，产生新的键值对作为中间结果**，然后MapReduce计算框架会自动将这些中间结果数据做**聚合处理**，然后经过shuffle将键**相同的数据分发给reduce函数处理**。reduce函数以键值对的形式对处理结果进行输出。要用表达式的形式表达的话，大致如下

~~~
{key1,value1}----->{key2,List<value2>}----->{key3,value3}
~~~

### MapReduce的运行环境

​	与HDFS相同的是，MapReduce计算框也是主从架构。支撑MapReduce计算框架的是**JobTracker**和**TaskTracker**两类后台进程。一个MapReduce 作业的计算工作都由TaskTracker 完成。用户向Hadoop 提交作业，**Job Tracker 会将该作业拆分为多个任务， 并根据心跳信息交由空闲的TaskTracker 启动。**Hadoop 集群中绝大多数服务器**同时运行 DataNode 进程和 TaskTracker 进程。**

![image-20210407200509173](C:\Users\ql\AppData\Roaming\Typora\typora-user-images\image-20210407200509173.png)

##### JobTracker进程

​	Job Tracker 在集群中扮演了主的角色，它主要负责**任务调度和集群资源监控**这两个功能，但并不参与具体的计算。一个Hadoop 集群只有一个JobTracker ，存在单点故障的可能，所以必须运行在相对可靠的节点上，一JobTracker 出错，将导致集群所有正在运行的任务全部失败。

​	与HDFS 的NameNode 和DataNode 相似， **TaskTracker 也会通过周期性的心跳向JobTracker汇报当前的健康状况和状态**，心跳信息里面包括了自身计算资源的信息、被占用的计算资源的信息和正在运行中的任务的状态信息。**JobTracker 则会根据各个TaskTracker 周期性发送过来的心跳信息综合考虑TaskTracker 的资源剩余量、作业优先级、作业提交时间等因素，为TaskTracker分配合适的任务。**

##### TaskTracker进程

​	TaskTracker 在集群中扮模了从的角色，它主要负责**汇报心跳和执行JobTracker 的命令**这两个功能。一个集群可以有多个TaskTracker ，但一个节点只会有一个TaskTracker ， 并且**TaskTracker和DataNode 运行在同一个节点之中**，这样， 一个节点既是计算节点又是存储节点。TaskTracker会周期性地将各种信息汇报给JobTracker ，而JobTracker 收到心跳信息， 会根据心跳信息和当前作业运行情况为该TaskTracker 下达命令， 主要包括启动任务、提交任务、杀死任务、杀死作业和重新初始化5 种命令。

### ![image-20210407200400892](C:\Users\ql\AppData\Roaming\Typora\typora-user-images\image-20210407200400892.png)

​	**分布式计算需要将不同服务器上的相关数据合并到一起进行下一步计算，这就是 shuffle。**	在 map 输出与 reduce 输入之间，MapReduce 计算框架处理数据合并与连接操作，这个操作有个专门的词汇叫 shuffle。

![image-20210407200536282](C:\Users\ql\AppData\Roaming\Typora\typora-user-images\image-20210407200536282.png)

​	每个 **Map 任务的计算结果都会写入到本地文件系统**，等 Map 任务全部快要计算完成的时候，MapReduce 计算框架会**启动 shuffle 过程**，在 Map 任务进程调用一个 Partitioner 接口，对 Map 产生的每个 进行 Reduce 分区选择，**然后通过 HTTP 通信发送给对应的 Reduce 进程**。这样不管 Map 位于哪个服务器节点，**相同的 Key 一定会被发送给相同的 Reduce 进程**。Reduce 任务进程对收到的 进行排序和合并，相同的 Key 放在一起，组成一个 传递给 Reduce 执行。