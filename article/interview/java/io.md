### BIO,NIO,AIO 有什么区别?

+ **BIO (Blocking I/O):** 同步阻塞 I/O 模式，数据的读取写入必须阻塞在一个线程内等待其完成。在连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。
+ **NIO (Non-blocking/New I/O):** NIO 是一种同步非阻塞的 I/O 模型，它支持面向缓冲，基于通道I/O 操作方法。 NIO 提供了与传统BIO 模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。阻塞模式与BIO基本一致，但性能和可靠性都不好；非阻塞模式正好与之相反。
+ **AIO (Asynchronous I/O):** AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步
  非阻塞的 IO 模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不
  会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。
