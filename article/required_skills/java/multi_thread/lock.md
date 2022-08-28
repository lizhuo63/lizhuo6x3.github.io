# 内部锁

+ synchronized

synchronized是同步阻塞，采用的是悲观并发策略，是JVM级别的。

# 显示锁

顶级接口 Lock，Lock是同步非阻塞，采用的是乐观并发策略。是API级别的，通过Lock可以知道有没有成功获取锁，可以分别定义读写锁提高多个线程读操作的效率。

**<u>锁的状态总共有 4种：无锁、偏向锁、轻量级锁和重量级锁。</u>**

## 常用方法

**API：**

+ **lock( )：**获取锁，死锁风险很高
+ **tryLock(long time, TimeUnit unit)：**在指定时间内，若当前锁对象已被其他线程持有或超时，则放弃申请该锁
+ **isHeldByCurrentThread()：**判断锁是否被当前线程所持有
+ **lockInterruptibly( )：**如果当前线程未被中断则获得锁，若被 **interrupt( )** 中断就报异常并终止当前线程。
+ **newContition( )：**该方法返回 Condition 对象，该类的await()、signal( )可实现等待/通知模式，需要持有Lock锁对象。创建多个Condition 对象可实现指定线程的等待通知，更加灵活。
  + **await( )：**令线程等待，并释放锁
  + **singal( )：**当前 Condition 对象的等待队列中唤醒 一个线程，尝试申请锁，成功后继续执行

**技巧：**

+ 释放锁

  ```java
  finally {
  		if (lock.isHeldByCurrentThread()){
  				lock.unlock();
  		}
  }
  ```

+ <u>ReentrantLock 配合使用 lockInterruptibly( )和 interrupt( ) 可有效避免死锁问题。</u>

## ReentrantLock (可重入锁)

可重入锁也叫作递归锁，指在同一线程中，在外层函数获取到该锁之后，内层的递归函数仍然可以继续获取该锁。ReentrantLock和synchronized都是可重入锁。可多次获取同一把锁，能够避免死锁。

```java
static Lock lock = new ReentrantLock();
public static void main(String[] args) {
     lock.lock();
     try {
          //获取本锁保护的资源
          System.out.println(Thread.currentThread().getName() + "开始执行任务");
         lock.lock();
     } finally {
            lock.unlock();
  		      lock.unlock();
     }
}
```

**常用API：**

+ int getHoldCount() 返回当前线程调用 lock()方法的次数
+ int getQueueLength() 返回正等待获得锁的线程预估数
+ int getWaitQueueLength(Condition condition) 返回与 Condition 条件相关的等待的线程预估数
+ boolean hasQueuedThread(Thread thread) 查询参数指定的线程是否在等待获得锁
+ boolean hasQueuedThreads() 查询是否还有线程在等待获得该锁
+ boolean hasWaiters(Condition condition) 查询是否有线程正在等待指定的 Condition 条件
+ boolean isFair() 判断是否为公平锁
+ boolean isHeldByCurrentThread() 判断当前线程是否持有该锁
+ boolean isLocked() 查询当前锁是否被线程持有；



## ReentrantReadWriteLock(读写锁)

java.util.concurrent.locks包中定义了ReadWriteLock接口，是读写锁的顶层接口。

​		ReentrantReadWriteLock是一种对synchronized和ReentrantLock这些排他锁的改进，读之前必须获取读锁，更新之前需获取写锁。它**<u>允许并发读串行写</u>**，写锁是排他锁，读锁是共享锁，不允许这两种锁被同时持有。用于保障多线程并发读取到最新的共享数据。读读共享, 读写互斥,写写互斥。

**注意：**readLock()与writeLock()方法返回的锁对象是同一个锁的两个不同的角色, 而不是分别获得两个不同的锁对象。即ReadWriteLock 接口实例可以充当两个角色。

```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();
//读数据
readLock.lock(); //申请读锁
try{
// 读取共享数据
}finally{
		readLock.unlock(); 
}
//写数据
writeLock.lock(); //申请写锁
try{
// 更新修改共享数据
}finally{
		writeLock.unlock(); 
}
```



## 公平锁与非公平锁

​	**公平锁：**优先将锁分配给排队时间最长的线程。在一个回合内，实现所有线程都获取一遍锁。`new ReentrantLock(true);`可获取一个公平锁。它需要维护一个有序队列，成本高性能低，默认是不采用的。

​	**非公平锁：** 在分配锁时不考虑线程排队中的等待情况，直接尝试获取锁，随即抢占。所以大多数锁都是非公平的。



## 共享锁和独占锁

**独占锁：**也叫互斥锁，每次只允许一个线程持有该锁，ReentrantLock为独占锁的实现。
**共享锁：**允许多个线程同时获取该锁，并发访问共享资源。ReentrantReadWriteLock中的读锁为共享锁的实现。



## 重量级锁和轻量级锁

**重量级锁：**监视器锁（Monitor）基于操作系统的互斥量（Mutex Lock）实现，而重量级锁是基于监视器锁实现的，会导致进程在用户态与内核态之间切换，相对开销较大，运行效率不高。

为了减少获取锁和释放锁所带来的性能消耗及提高性能，引入了轻量级锁和偏向锁。
**轻量级锁：**轻量级锁的核心设计是在没有多线程竞争的前提下，减少重量级锁的使用以提高系统性能。轻量级锁适用于线程交替执行同步代码块的情况（即互斥操作），如果同一时刻有多个线程访问同一个锁，则将会导致轻量级锁膨胀为重量级锁。

**偏向锁：**【是对可重入锁的优化】。偏向锁主要要目的是在同一个线程多次获取某个锁的情况下消除这个线程锁重入的开销，因为轻量级锁的获取及释放需要多次CAS原子操作，而偏向锁只需要在切换ThreadID时执行一次CAS原子操作，因此可以提高锁的运行效率。在出现多线程竞争锁的情况时，JVM会自动撤销偏向锁。



## 分段锁

分段锁并非一种实际的锁，而是一种思想，用于将数据分段并在每个分段上都单独加锁，把锁进一步细粒度化，以提高并发效率。ConcurrentHashMap在内部就是使用分段锁实现的。