# 基本概念

## 进程&线程

+ **进程：**进程是程序实体的运行过程（创建，运⾏到消亡）是资源分配的基本单位。它可异步并发执行。它的本体是一个**“进程控制块”(PCB)**的数据结构，它常驻内存，由程序(可共享)，数据，PCB共同构成**进程实体**，PCB是进程的唯一标志。以下是PCB主要的包含信息。

  ​	<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_20-55-05.png" style="zoom: 80%;" />

+ **线程：**线程(不具备结构之外的资源，内存占用小)是进程的分支(子任务)、“轻量级进程”，是调度的基本单位(CPU执行的最小单元)。由线程ID、程序计数器，寄存器集合和堆栈构成，是进程中的一个实体，进程下的所有线程都共享进程的全部资源。但是不同进程之间的线程是互不可见的。

  ​	<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_21-12-50.png" style="zoom:80%;" />

## 线程状态

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_21-17-19.png)

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_21-17-39.png)

## 并发三要素

+ 原子性：指一组操作要么全部执行成功要么全部执行失败。

+ 可见性：当线程a对共享变量进行修改，线程b能够立刻发现。（synchronized,volatile）
+ 有序性：程序执行的顺序按照代码的先后顺序执行。（处理器会对指令进行优化时可能会对指令进行重排序）

**出现线程安全问题的原因：**

- 线程切换带来的原子性问题 ==》JDK Atomic开头的原子类、synchronized、LOCK
- 缓存导致的可见性问题 ==》synchronized、volatile、LOCK，可以解决可见性问题
- 编译优化带来的有序性问题 ==》Happens-Before 规则可以解决有序性问题

# Java锁机制

| 序号 |  锁名称  |                             应用                             |
| :--: | :------: | :----------------------------------------------------------: |
|  1   |  乐观锁  |                             CAS                              |
|  2   |  悲观锁  |        synchronized、Reentrantlock、vector、hashtable        |
|  3   |  自旋锁  |                             CAS                              |
|  4   | 可重入锁 |              synchronized、Reentrantlock、Lock               |
|  5   |  读写锁  | ReentrantReadWriteLock，CopyOnWriteArrayList、CopyOnWriteArraySet |
|  6   |  公平锁  |                     Reentrantlock(true)                      |
|  7   | 非公平锁 |              synchronized、reentrantlock(false)              |
|  8   |  共享锁  |                 ReentrantReadWriteLock中读锁                 |
|  9   |  独占锁  | synchronized、vector、hashtable、ReentrantReadWriteLock中写锁 |
|  10  | 重量级锁 |                         synchronized                         |
|  11  | 轻量级锁 |                          锁优化技术                          |
|  12  |  偏向锁  |                          锁优化技术                          |
|  13  |  分段锁  |                      concurrentHashMap                       |
|  14  |  互斥锁  |                         synchronized                         |
|  15  |  同步锁  |                         synchronized                         |
|  16  |   死锁   |                      相互请求对方的资源                      |
|  17  |  锁粗化  |                          锁优化技术                          |
|  18  |  锁消除  |                          锁优化技术                          |

## 乐观锁

认为遇到并发写的概率比较低，读数据时认为别的线程不会对它进行修改（所以没有上锁）。

**CAS：**比较并替换，将当前值（主内存中的值）与预期值（当前线程中的值）进行比较，一样则更新，否则失败重试（自旋）。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-07_21-24-53.png)

## 悲观锁

认为遇到并发写的可能性高，所以每次读写数据时都会上锁。其他线程想要读写这个数据时，会被这个线程block，直到这个线程释放锁然后其他线程获取到锁。

## 自旋锁

线程等待，让无锁的线程执行一个忙循环（自旋），而不是挂起切换这个线程。

+ 优点：避免了线程切换的开销。
+ 缺点：如果自旋时间过长，就会白白消耗CPU资源

## 可重入锁（递归锁）

 持锁线程能够再次获取该锁而不会被锁阻塞。能够有效避免死锁。

## 读写锁

通过`ReentrantReadWriteLock`类来实现。可实现并发读，串行写。

