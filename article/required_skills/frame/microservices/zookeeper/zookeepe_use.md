# 简介

Zookeeper是一个开源的分布式的，基于观察者模式设计的分布式服务管理框架 。它负责 
存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper。上注册的那些观察者做出相应的反应。 简而言之。Zookeeper 就是一个文件系统+通知机制。

## 特点

![image-20220730092736351](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220730092736351.png)

+ Zookeeper:一个领导者(Leader)，多个跟随者(Follower)组成的集群。 
+ 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。所以Zookeeper适合安装奇数台服务器，避免服务器的浪费
+ 全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。 
+ 更新请求顺序执行，来自同一个Client的更新请求按其发送顺序依次执行。 【请求队列，先进先出】
+ 数据更新原子性，一次数据更新要么成功，要么失败。 【天然事务支持】
+ 实时性，在一定时间范围内，Client能读到最新数据。 

## 使用场景

+ **统一命名服务：**服务名映射多台服务IP
+ **统一配置管理：**快速同步分布式服务配置，通过将配置信息写入ZooKeeper上的一个Znode，并让各个客户端服务器监听这个Znode来实现。 
+ **统一集群管理：**根据节点的实时信息，调整服务状态。
+ **软负载均衡：**实时让请求线程最少的服务器去处理当前的新请求。



# 安装

以测版本 apache-zookeeper-3.6.3-bin.tar.gz

## 配置

```bash
# 服务的通信心跳时间
tickTime=2000
# 主节点和从节点初始化的超时时间 n*tickTime ,以下是20秒
initLimit=10 
# 主节点和从节点初始化后的心跳超时时间 n*tickTime ,以下是10秒
syncLimit=5
# 数据存储目录
dataDir
# 客户端连接端口
clientPort 
```

## 集群搭建

1. 克隆虚拟机

2. 清空对应克隆机的data和logs下的文件（随便）

3. 在data下新建 myid 文件；只需要填入id即可，如 `1`

4. 在conf下添加

   ```bash
   # ip注意变更
   server.1=192.168.154.135:2888:3888
   server.2=192.168.154.136:2888:3888
   server.3=192.168.154.137:2889:3889
   ```
5. 重新启动，注意需启动多个节点，zookeeper才会启动成功，即查看状态时会显示 `Mode: leader `或者 `Mode: follower`。

## 数据结构

ZooKeeper数据模型的结构与Unix文件系统很类似，是一个树形结构，每个节点称做一个ZNode。每一个ZNode默认能够存储IMB的数据，每个Node都可以通过其路径唯一标识。节点可以分为四大类： 

+ PERSISTENT持久化节点 
+ EPHEMERAL临时节点：-e 
+ PERSISTENT SEQUENTIAL持久化顺序节点：-s 
+ EPHEMERAL SEQUENTIAL临时顺序节点：-es 

## 常用命令

+ 连接服务器：`/zkCli.sh  -server ip:port`

+ 断开连接：`quit`

+ 查看帮助：`he;p`

+ 显示目录下的所有节点：`ls`

+ 创建节点：

  + 持久化节点：`create /节点路径 value`
  + 临时节点：`create -e /节点路径 value`
  + 顺序节点：`create -s /节点路径 value`

+ 设置节点值：`set/节点路径 value`

+ 删除单节点：`delete /节点路径`

+ 删除指定父节点：`deleteall /节点路径`

+ 获取节点值：`get /节点路径`

  ![image-20220730102745364](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220730102745364.png)



## curatorApi 操作

**环境依赖**

```xml
 <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-framework</artifactId>
      <version>4.3.0</version>
    </dependency>
    <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-recipes</artifactId>
      <version>4.3.0</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.36</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.7.32</version>
    </dependency>
```

+ **连接**

  ```java
  RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
  		client = CuratorFrameworkFactory.builder().connectString("192.168.154.131:2181")
  				.sessionTimeoutMs(5000).retryPolicy(retryPolicy).namespace("work").build();
  		client.start();
  ```

+ 增删改方法见名知意。