# 进程是什么

## 作业&进程？？？

​		**进程**本身就不是程序，程序只是一个实体，是硬盘上的一个可执行文件。实际上，进程是程序在整个运行过程中的实体，是程序的执行过程，是分配资源的基本单位，每一个进程都可以用进程控制块【**PCB**】来描述。

​		**作业**是要求计算机所做工作的集合。一个作业的完成要经过作业的提交、收容、执行和完成4个阶段。而进程是已提交的程序所执行过程的描述。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-16_20-57-41.png ':size=20%')

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-17_08-50-50.png)

+ **进程状态：**标记当前的进程状态，包括以上五种
+ **进程编号：**进程的唯一标识
+ **程序计数器：**指向下一条指令的地址
+ **CPU寄存器：**它的数量和类型不等，在运行中断时它们会与程序计数器一起保存，可实现后续执行。
+ **CPU调度信息：**包括进程优先级，调度队列的指针，其他调度参数。
+ **内存管理信息：**包括基地址和界限寄存器的值，页表，段表。
+ **记账信息：**包括CPU时间，时间使用时间，时间期限，记账数据，作用和进程数量
+ **IO状态信息：**包括分配给进程的IO设备列表，打开文件列表等

## 进程的创建

​		当系统启动后，根进程 **init** (pid总为1)就会被创建，此外根进程能以树的形式创建各种子进程。<u>子进程所需的资源可从操作系统或父进程那里获取，父进程可在子进程之间分配资源以及共享资源。</u>父进程可与子进程并发执行，也有可能需要等待某个或所有子进程执行完毕。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-17_09-49-41.png)

## 进程的终止

​		当进程执行完毕后可由父进程通过系统调用 exit( ) 可终止进程，此时该进程所持有的所有资源都将由操作系统释放掉，但该子进程在进程表（记录了每个进程的状态以及资源）中的条目还在【称为：僵尸进程】，原因是父进程还不确定子进程是否终止成功，这时父进程可通过系统调用 wait( ) 获取子进程的退出状态，并拿掉它在进程表中的记录。此时子进程就终止完毕了。

# 进程调度

+ **作业队列：**包含系统内的所有进程
+ **就绪队列：**包含驻留内存的，就绪的，等待运行的所有进程
+ **设备队列：**包括临时需要请求IO设备的进程

​		程序进入系统时会以进程的形式被加入到作业队列，随后转到就绪队列准备执行。

​	![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-17_09-36-06.png)

# 进程通信

​		某些任务需要多个进程协作完成，此时他们就需要一种进程间的通信机制来完成数据的交换以及信息的共享。常有共享内存和消息传递种模式：

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-17_10-20-14.png)

## 共享内存

​		通常操作系统会禁止一个进程去访问另一个进程，共享内存机制需要二者合力打破限制，通过在共享区域读写数据来交换消息，信息不受操作系统控制，该机制确保各个进程不同时向同一个位置写数据。

​		多用于本机。

## 消息传递

​		主要用于解决生产者和消费者的通信问题，各进程通过发送和接收消息来实现通信。多用于分布式。

​		**缓存队列（消息队列）：**用于存储通信的消息

​		有以下通信模式：

​		**直接通信：**需要明确每个通信进程指定生产者和消费者

​		**间接通信：**通过邮箱或者端口来发送、接收消息。邮箱可以存，也可以删，具有唯一标识。

​		**同步通信：**发送时阻塞，知道消息被接收。接收时阻塞，直到成功接收且有消息可用。

​		**异步通信：**发送时可恢复操作，接收时可允许接收空消息或无效消息。

### 套接字

是为通信的端点，由IP和Port组成，它是分布式通信中一种低级形式，只支持无结构的字节流。

1. 服务器监听指定端口，等待连接
2. 客户端发出连接请求
3. 服务器接受连接，未客户端分配一个端口，建立通信通道

### 远程过程调用RPC

​	RPC通信的消息具有明确的结构且允许客户端调用远程主机

### 管道

**匿名管道：**只支持父子进程间的通信，只支持单向的通信。

**命名管道：**支持任意进程简单通信，而且支持双向通信。

