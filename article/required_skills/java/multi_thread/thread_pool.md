# 线程池简介

线程池内部可以预先创建一定数量的工作线程，用户直接将任务作为一个对象提交给线程池, 线程池将这些任务缓存在工作队列中, 线程池中的工作线程不断地从队列中取出任务并执行。

**优点：**

+ **降低资源消耗**。能够重复利用已创建的线程，降低了线程频繁创建和销毁造成的消耗。
+ **提高响应速度**。当任务到达时可越过线程的创建，能够立即执行。
+ **提高线程的可管理性**。使用线程池可以进行统一的资源分配，调优和监控。

# 线程池创建

## 原生构建

```java
public ThreadPoolExecutor(
  	 //核心线程数。当线程池接收到新任务，且当前工作线程数少于corePoolSize时，即使其他工作线程处于空闲状态，也会创建一个新线程来处理该请求，直到线程数达到corePoolSize。
    int corePoolSize,
  
  	//最大线程数。若活跃线程数低于该值且所有线程均繁忙，也会创建新线程。当活跃线程数达到该数值后，后续任务将会阻塞。当maximumPoolSize被设置为无界值（如Integer.MAX_VALUE）时，线程池可以接收任意数量的并发任务。通过设置corePoolSize和maximumPoolSize相同，可以创建一个固定大小的线程池。
    int maximumPoolSize,
  
  	//线程闲置时长。超过该时长的空闲和非Core线程会被回收。
    long keepAliveTime,
  
	  //指定 keepAliveTime 参数的时间单位。
    TimeUnit unit,
  
  	//任务队列。在阻塞队列为空时会阻塞当前线程的元素获取操作。用于存储提交但未执行的任务，线程池可以接收任意数量的并发任务。
  	//ArrayBlockingQueue:基于数组的有界阻塞队列，必须设置大小，按FIFO排序
  	//LinkedBlockingQueue:基于链表的有界阻塞队列，容量可设可不设，但不设会有资源耗尽的危险。按FIFO排序，
  	//PriorityBlockingQueue:是具有优先级的无界队列。
  	//SynchronousQueue:同步队列,不存储元素,任务不会被保存。在没有空闲线程的条件下，来一个任务就创建一个线程，总是将任务提交给线程执行。
  	//DelayQueue:无界阻塞延迟队列,队列中每个元素都有过期时间，当从队列获取元素（元素出队）时，只有已经过期的元素才会出队，队列头部的元素是过期最快的元素。
    BlockingQueue<Runnable> workQueue,
  
  	//线程工厂。用于指定为线程池创建新线程的方式。可以更改所创建的新线程的名称、线程组、优先级、守护进程状态等。Executors为线程池工厂类，用于快捷创建线程池（Thread Pool）；ThreadFactory为线程工厂类，用于创建线程（Thread）。
    ThreadFactory threadFactory,
  
  	//拒绝策略。当达到最大线程数或线程池关闭时如何拒绝
  	//AbortPolicy:默认策略-拒绝新任务并且抛RejectedExecutionException异常。
  	//DiscardPolicy:拒绝新任务不抛异常
  	//DiscardOldestPolicy:抛弃最老任务
  	//CallerRunsPolicy:若添加失败，则由任务提交线程自己去执行
  	//自定义拒绝策略:实现RejectedExecutionHandler接口的rejectedExecution方法
    RejectedExecutionHandler handler
) 

//========调度器的钩子方法:
//任务执行之前的钩子方法（前钩子）。可用于重新初始化ThreadLocal线程本地变量实例、更新日志记录、开始计时统计、更新上下文变量等。
beforeExecute(Thread t, Runnable r)
//任务执行之后的钩子方法（后钩子）。可用于清除ThreadLocal线程本地变量、更新日志记录、收集统计信息、更新上下文变量等。
afterExecute(Runnable r, Throwable t)
//:线程池终止时的钩子方法（停止钩子）
terminated()
[注意]
beforeExecute和afterExecute两个方法在每个任务执行前后被调用,如果钩子(回调方法)引发异常,内部工作线程可能失败并突然终止
```

## 任务执行流程

![image-20220913161137710](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220913161137710.png)

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220913121255971.png" alt="image-20220913121255971" style="zoom:67%;" />

Excutor框架是JDK提供的访问线程池的框架：

## 4个线程池模版

+ **newCachedThreadPool ：**创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程。若来了新任务而所有线程均繁忙，就会新建线程（线程最大并发数不可控制，依赖于OS或者JVM）。若当前线程的数量超过了处理的任务数量，就会回收空闲（60秒不执行任务）线程。**适用于快速处理突发性强、耗时较短的任务场景**
+ **newFixedThreadPool：**创建一个固定数量的线程池，可控制线程最大并发数，超出的线程会在队列中等待。如果线程数没有达到“固定数量”，每次提交一个任务线程池内就创建一个新线程，直到线程达到线程池固定的数量。一旦达到“固定数量”就会保持不变，新任务会进入阻塞队列中（无界的阻塞队列）。如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。**适用于处理CPU密集型的任务**
+ **newScheduledThreadPool ：** 创建一个定时线程池，支持延时及周期性地调度执行任务。**适用于周期性地执行任务的场景。**
+ **newSingleThreadExecutor ：**创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。其阻塞队列是无界的，**任务是按照提交的次序顺序执行的**。

# 线程池的5种状态

**RUNNING**：线程池创建之后的初始状态即运行状态，这种状态下可以执行任务。
**SHUTDOWN**：shutdown()之后步入该状态，将不再接受新任务，但是会把工作队列中的任务执行完毕。
**STOP**：shutdownNow()之后步入该状态，线程池将不再接受新任务，也不会处理工作队列中的剩余任务，并且将会中断所有工作线程。
**TIDYING**：当所有任务都已终止或者处理完成会步入此状态，准备执行terminated()钩子方法。
**TERMINATED**：执行完terminated()钩子方法之后的状态。

### 示例

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

**提交任务：**

+ submit()；可接收Callable[可返回执行结果、可抛异常]、Runnable[无返回结果、不可抛异常]两种类型的参数。submit()提交任务后可返回Future对象【封装有执行结果和异常】，可通过它获取结果。

+ execute()；只能接收Runnable[无返回结果、不可抛异常]。execute()提交任务后会没有返回值

**关闭线程池：**

+ shutdown()；令线程池状态会变为SHUTDOWN，直到线程池中的任务都已经处理完成才会退出。此时线程池将拒绝新任务，不能再往线程池中添加新任务，否则会抛出RejectedExecutionException异常。

+ shutdownNow()；线程池状态会立刻变成STOP，试图停止所有正在执行的线程，并且不再处理还在阻塞队列中等待的任务，会返回那些未执行的任务。

































