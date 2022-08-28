# 线程是什么

​		之前说到，进程是程序运行的实体。那么一个程序被设计出来一定是希望可以方便地解决某些问题，高效地完成任务。这就需要将批量任务分发下去，交给底下的人协同处理，也就是线程了，线程是CPU资源使用的最小单位，它拥有线程ID，程序计数器，寄存器组和堆栈，能够与同进程下的其他线程共享内存和资源。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-17_14-24-54.png)



# 多线程模型

<u>**CPU资源是直接分配给内核线程的，间接映射到用户线程。**</u>

**用户线程：**位于内核之上，无需内核提供管理

**内核线程：**由操作系统直接管理

#### 多对一

优点：切换效率高、缺点：并发度差

![image-20220618015037350](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220618015037350.png)

#### 一对一

优点：并发度高、缺点：切换效率低

![image-20220618015116038](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220618015116038.png)

#### **多对多**

同时保证了并发度和切换效率。

![image-20220618015226953](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220618015226953.png)

# java线程