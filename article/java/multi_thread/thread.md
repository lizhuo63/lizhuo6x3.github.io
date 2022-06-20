# 线程的创建

java中线程的本质就是从当前运行分支中创建出一个新的**可运行的**分支。所以具体有如下方案：

+ **重写Thread类的run()方法**

  <u>注意使用start( )方法调用，调用run( )方法不会新建线程</u>

  ```java
  public class NewThread extends Thread{
      @Override
      public void run() {
          System.out.println("子线程执行");
      }
      public static void main(String[] args) {
          NewThread thread = new NewThread();
          thread.start();
      }
  }
  ```

+ **重写Runnable接口的run()方法**

  <u>注意使用start( )方法调用，调用run( )方法不会新建线程</u>

  ```java
  public class NewThread {
      public static void main(String[] args) {
          Runnable runnable= () ->{
              System.out.println("子线程执行");
          };
          new Thread(runnable).start();
      }
  }
  ```

+ **重写Callable的call( )方法**

  ```java
  public class NewThread {
      public static void main(String[] args) {
          Callable callable = () -> {
              System.out.println("子线程执行");
              return null;
          };
          //FutureTask可包装Runable和Callable
          FutureTask futureTask = new FutureTask<>(callable);
          futureTask.run();
      }
  }
  ```

+ **线程池**

  **有如下4中线程池模式：**

  + **newCachedThreadPool ：**创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，否则新建线程。（线程最大并发数不可控制）
  + **newFixedThreadPool：**创建一个固定大小的线程池，可控制线程最大并发数，超出的线程会在队列中等待。
  + **newScheduledThreadPool ：** 创建一个定时线程池，支持定时及周期性任务执行。
  + **newSingleThreadExecutor ：**创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

  <u>注意并发任务执行完毕后应当关闭线程池</u>

  ```java
  public class NewThread {
      public static void main(String[] args) {
          ExecutorService executorService = Executors.newCachedThreadPool();
          //提交可执行代码，可包装Runable和Callable【可使用lamda表达式】
          executorService.submit(new Runnable() {
              @Override
              public void run() {
                  System.out.println("子线程执行");
              }
          });
          executorService.shutdown();
      }
  }
  ```




# 线程状态

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-18_18-28-39.png)

+ **NEW,新建状态。** 创建了线程对象,在调用 start()启动之前的状态;
+ **RUNNABLE,可运行状态。** 包含:READY 和RUNNING 两个状态. READY 状态可以被线程调度器进行调度并进入RUNNING 状 态 ,Thread.yield()方法可从RUNNING状态转换为 READY 状态。
+ **BLOCKED阻塞状态。**线程发起阻塞的 I/O 操作或者申请由其他线程占用的独占资源（如锁）时会转入BLOCKED 阻塞状态.。处于阻塞状态的线程不会占用CPU资源，当阻塞I/O操作执行完,或者线程获得了其申请的资源,线程可以转换为 RUNNABLE。
+ **WAITING 等待状态。** object.wait(), thread.join()会把线程转换为 WAITING 等待状态, 执行 object.notify()方法,或者加入的线程执行完毕,当前线程会转换为 RUNNABLE 状态。
+ **TIMED_WAITING 状态。**与 WAITING 状态的区别在于处于该状态的线程不会无限的等待,如果该线程在指定的时间内没有完成期望的操作，就会自动转换为 RUNNABLE。
+ **TERMINATED 终止状态。**线程结束或运行完毕。

# 多线程风险和特性

+ **线程安全问题：**多线程的执行顺序是随机的，若并发访问控制不当，就会产生数据一致性问题。主要是由硬件层的内存缓冲区和软件层的JIT编译器的指令重排引起的。**<u>实现线程安全性必须要为共享数据的读和写同时加上同一把锁。</u>**
  
  + **原子性：**可通过锁和CAS指令实现
    + 其他线程只能看到共享变量更改前和更改完毕之后的结果，中间过程值不可见
    + 一组指令同时成功或失败
  + **可见性：**
    + 对共享变量进行更改后，其他线程能立即获得这个新数据
  + **有序性：**序 ==》 内存访问操作的顺序。可以使用 volatile、synchronized 关键字实现有序性
  
+ **线程活性问题：**线程一直处于非可运行状态。
  
  + **死锁：**互相占有独占资源
  
    当线程需要多个锁资源时，保持锁资源顺序一致，可有效解决死锁。
  
  + **锁死：**线程无法唤醒致任务无法进展
  
  + **活锁：**线程一直在做无用功，任务一直无法进展
  
  + **饥饿：**需要的资源一直被占用而无法释放
  
    

# java内存模型

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-19_10-17-38.png)

+ 方法执行依赖于CPU，但CPU从不直接从主内存中取数据，而是通过缓存，
+ CPU不能访问其他CPU的寄存器，但能通过缓存一致性协议访问其他CPU的缓存
+ 主内存中的共享数据被A核处理后未刷入缓冲区，若在该数据被刷入缓存之前被B核访问，就会**发生可见性问题**，获取到脏数据。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-19_10-46-59.png)

+ 线程间的共享数据存在主内存
+ 工作内存是线程私有的，它是抽象的，包含上图的缓冲区和寄存器。它存储的是共享数据的副本，且对其他线程不可见

# 线程同步

​		**<u>多线程的并发：一个CPU核心在任意时刻只能为一个线程提供时间片，但是在任意短的时间段内是可以为多个线程提供资源的；在如今多CPU时代下，宏观上是完全有可能同时为多个线程提供时间片的，即并行服务的支持。所以可以认为，多核下的线程是可以同时执行的。</u>**

​		线程同步机制是一套用于协调线程之间的数据访问的机制，用于保障线程安全。Java 平台提供解决方案包括: 锁, volatile 关键字, final 关键字,static 关键字,以及相关的线程调度API,如wait()，notify()等

​		<u>**java线程在执行过程中若出现了异常，会立即中断并释放锁对象，且不会干扰其他线程执行。**</u>

## 锁

​		锁是一种资源，是访问共享数据前的必须条件。持有锁的线程才被允许访问共享数据，访问结束后应当立即释放锁。

+ **原子性：**锁具备互斥性，即一个锁只能同时被一个线程所持有，并以此来保障线程的原子性。
+ **可见性：**在java中，获取锁时会自带刷新CPU缓存，释放锁时会自带将新数据刷入CPU缓存，并以此来保障线程的可见性。
+ **有序性：**

 **锁的可重入性：**

​		指持有该锁的线程可否继续申请该锁。如A( )调用B( ),且A()、B()同时需要a锁，此时若a锁不具备可重入性，程序就会锁死，若a锁具备可重入性，程序就可继续运行。

## synchronized

​		一种互斥锁，可以保证原子性，也可以保证可见性。粒度越细，执行效率越高。

```java
public class NewThread {
    public static void main(String[] args) {
        NewThread o = new NewThread();
        new Thread(() -> {
            o.mm();
        }, "A").start();
        new Thread(() -> {
            o.mm();
        }, "B").start();
    }

    public void mm() {
        synchronized (this) {
            for (int i = 0; i < 100; i++) {
                System.out.println(Thread.currentThread().getName() + " ==== " + i);
            }
        }
    }
}
```

## volatile

​		修饰变量，可令变量的值直接基于主内存读写，仅能够保障可见性。异步非阻塞，效率高

## CAS（比较交换算法）

​		以 i++ 为例，一次循环操作涉及从主内存取值，计算，刷入主内存。CAS的核心在于在刷新主内存之前或获取当前内存中的 i 的值，如果它 和 当前线程 工作内存中的 i 一致 ，即目前无其他线程对 i 进行干扰，才会将新值刷入主内存。

## 原子变量类

​		原子变量类基于CAS实现的, 当对共享变量进行read-modify-write。更新操作时,通过原子变量类可以保障操作的原子性与可见性.

+ 基础数据型 AtomicInteger, AtomicLong, AtomicBoolean
+ 数组型 AtomicIntegerArray,AtomicLongArray,AtomicReferenceArray
+ 字段更新 AtomicIntegerFieldUpdater,AtomicLongFieldUpdater,AtomicReferenceFieldUpdater
+ 引用型 AtomicReference, AtomicStampedReference,AtomicMarkableReference

# 线程通信
