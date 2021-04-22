---
layout:     post
title:      ConcurrentHashMap源码
subtitle:   概述
date:       2021-03-06
author:     转载
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - 多线程
---

## ConcurrentHashMap 1.7

原文地址：https://mp.weixin.qq.com/s/AHWzboztt53ZfFZmsSnMSw  

###  存储结构

![image-20210422133616043](../../../../AppData/Roaming/Typora/typora-user-images/image-20210422133616043.png)

Java 7 中 ConcurrentHashMap 的存储结构如上图，ConcurrnetHashMap 由很多个 Segment  组合，而**每一个 Segment 是一个类似于 HashMap 的结构**，所以每一个 HashMap 的内部可以进行扩容。但是 **Segment 的个数一旦初始化就不能改变**，默认 Segment 的个数是 16 个，你也可以认为 ConcurrentHashMap 默认支持最多 16 个线程并发。 **Segment 数组 + HashEntry 数组 + 链表**

**Segment 是 ConcurrentHashMap 的一个内部类，主要的组成如下：**

![image-20210422140444979](../../../../AppData/Roaming/Typora/typora-user-images/image-20210422140444979.png)



Segment 继承了 ReentrantLock，所以 **Segment 是一种可重入锁，扮演锁的角色。**Segment 默认为 16，也就是并发度为 16。

**存放元素的 HashEntry，也是一个静态内部类，主要的组成如下：**

![image-20210422140551313](../../../../AppData/Roaming/Typora/typora-user-images/image-20210422140551313.png)



其中，用 volatile 修饰了 HashEntry 的数据 value 和 下一个节点 next，保证了多线程环境下数据获取时的**可见性**！

### JDK 1.7存取

![image-20210422142505976](../../../../AppData/Roaming/Typora/typora-user-images/image-20210422142505976.png)

先定位到相应的 Segment ，然后再进行 put 操作。

源代码如下：

![image-20210422142553300](../../../../AppData/Roaming/Typora/typora-user-images/image-20210422142553300.png)

首先会尝试获取锁，如果获取失败肯定就有其他线程存在竞争，则利用 `scanAndLockForPut()` 自旋获取锁。

1. 尝试自旋获取锁。
2. 如果重试的次数达到了 `MAX_SCAN_RETRIES` 则改为阻塞锁获取，保证能获取成功。





## ConcurrentHashMap 1.8

### 存储结构

![image-20210422134221592](../../../../AppData/Roaming/Typora/typora-user-images/image-20210422134221592.png)

在数据结构上， JDK1.8 中的ConcurrentHashMap 选择了与 HashMap 相同的**Node数组+链表+红黑树**结构；在锁的实现上，抛弃了原有的 Segment 分段锁，**采用`CAS + synchronized`实现更加细粒度的锁。**

**将锁的级别控制在了更细粒度的哈希桶数组元素级别，也就是说只需要锁住这个链表头节点（红黑树的根节点），就不会影响其他的哈希桶数组元素的读写，大大提高了并发度。**

### JDK 1.8存取

大致可以分为以下步骤：

1. 根据 key 计算出 hash 值，判断数组是否为空；
2. 如果是首节点，就直接返回；
3. 如果是红黑树结构，就从红黑树里面查询；
4. 如果是链表结构，循环遍历判断。

![image-20210422142740868](../../../../AppData/Roaming/Typora/typora-user-images/image-20210422142740868.png)

写入：

1. 根据 key 计算出 hashcode 。
2. 判断是否需要进行初始化。
3. 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
4. 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
5. 如果都不满足，则利用 synchronized 锁写入数据。
6. 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。

### ConcurrentHashMap 的 get 方法是否要加锁，为什么？

get 方法不需要加锁。因为 Node 的元素 value 和指针 next 是用 volatile 修饰的，在多线程环境下线程A修改节点的 value 或者新增节点的时候是对线程B可见的。

这也是它比其他并发集合比如 Hashtable、用 Collections.synchronizedMap()包装的 HashMap 效率高的原因之一。

### **JDK1.7 与 JDK1.8 中ConcurrentHashMap 的区别？**

- 数据结构：取消了 Segment 分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。
- 保证线程安全机制：JDK1.7 采用 Segment 的分段锁机制实现线程安全，其中 Segment 继承自 ReentrantLock 。JDK1.8 采用`CAS+synchronized`保证线程安全。
- 保证线程安全机制：JDK1.7 采用 Segment 的分段锁机制实现线程安全，其中 Segment 继承自 ReentrantLock 。JDK1.8 采用`CAS+synchronized`保证线程安全。
- 锁的粒度：JDK1.7 是对需要进行数据操作的 Segment 加锁，JDK1.8 调整为**对每个数组元素加锁（Node）**
- JDK1.8  Node 是类似于一个 HashEntry 的结构。它的冲突再达到一定大小时会转化成红黑树，在冲突小于一定数量时又退回链表。

### ConcurrentHashMap 和 Hashtable 的效率哪个更高？为什么？

ConcurrentHashMap 的效率要高于 Hashtable，因为 Hashtable 给整个哈希表加了一把大锁从而实现线程安全。而ConcurrentHashMap 的锁粒度更低，在 JDK1.7 中采用分段锁实现线程安全，在 JDK1.8 中采用`CAS+synchronized`实现线程安全。

### JDK1.8 中为什么使用内置锁 synchronized替换 可重入锁 ReentrantLock？

- 在 JDK1.6 中，对 synchronized 锁的实现引入了大量的优化，并且 synchronized 有多种锁状态，会从无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁一步步转换。
- 减少内存开销 。假设使用可重入锁来获得同步支持，那么每个节点都需要通过继承 AQS 来获得同步支持。但并不是每个节点都需要获得同步支持的，只有链表的头节点（红黑树的根节点）需要同步，这无疑带来了巨大内存浪费。

## 总结

Java7 中 ConcruuentHashMap 使用的分段锁，也就是每一个 Segment 上同时只有一个线程可以操作，每一个 Segment 都是一个类似 HashMap 数组的结构，它可以扩容，它的冲突会转化为链表。但是 Segment 的个数一但初始化就不能改变。

Java8 中的 ConcruuentHashMap  使用的 Synchronized 锁加 CAS 的机制。结构也由 Java7 中的 **Segment 数组 + HashEntry 数组 + 链表** 进化成了  **Node 数组 + 链表 / 红黑树**，Node 是类似于一个 HashEntry 的结构。它的冲突再达到一定大小时会转化成红黑树，在冲突小于一定数量时又退回链表。

有些人可能对 Synchronized 的性能存在疑问，其实 Synchronized 锁自从引入锁升级策略后，性能不再是问题，有兴趣的同学可以自己了解下 Synchronized 的**锁升级**。

- **实现线程安全的方式（重要）：** ① **在 JDK1.7 的时候，`ConcurrentHashMap`（分段锁）** 对整个桶数组进行了分割分段(`Segment`)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。（JDK1.6 以后 对 `synchronized` 锁做了很多优化）** 整个看起来就像是优化过且线程安全的 `HashMap`，虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本；② **`Hashtable`(同一把锁)** :使用 `synchronized` 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。