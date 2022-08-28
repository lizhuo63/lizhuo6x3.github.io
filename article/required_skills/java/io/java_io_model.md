# IO模型

![image-20220704134019601](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220704134019601.png)

![image-20220709064914990](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220709064914990.png)

数据的输入需要输入流从数据源头不断地读取数据，并载入**缓冲区**【系统内核的页缓存】；数据的输出需要输出端获取输出流，从页缓存不断的取数据，刷入输出端，**【注意】：输出流有丢数据的风险，需要用户强制刷写数据到磁盘 out.flush()**。flush是一个接口，在传统的io流【FileOutputStream】和getChannel( ).map【mmap内存映射,会得到一个内存空间，能减少内核的io调用】里没有具体实现，所以对于传统的io流即使out.flush()也相当于do nothing。而NIO new out.force() 会有具体实现。

**<u>Input会将数据先写到到内存缓冲区的页里面</u>**，没能及时持久化的页称之为脏页。系统内核有监听机制，当脏页占比、内核待机时间达到阈值时或机器正常关机时，会由系统内核自动执行持久化操作，将脏页写入磁盘。实际使用过程中传统io并没有丢数据，正是因为电脑开机时间足够长，是内核将数据持久化的，而不是传统 IO.flush( )的作用。







## BIO(Block IO)

```java
// Server:服务端
		ServerSocket server = new ServerSoket(port);
		while(true){
				new Thread(new SocketTask(server.accept())).start();//server.accept()会接收客户端，当没有用户访问时，线程会一直阻塞。
		}

// ==================================================

```

## NIO(NOBlock IO)

server.accept()会接收客户端，直接返回一个client对象，不会阻塞。随后立即对该对象进行判断是否为空，并进行后续退出或服务。

## Select、Epoll 多路复用

传统 **IO** 和 **BIO** 需要为每个Socket都建立一个单独的线程来处理该Socket上的数据，而 **多路复用I/O模型 **中只需一个线程就可以管理多个Socket，大大节省了线程损耗。

并且在真正有Socket读写事件时才会使用操作系统的I/O资源，大大节约了系统资源。

