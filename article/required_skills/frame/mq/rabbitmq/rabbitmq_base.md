# 安装

```bash
# 1.安装依赖
sudo yum install build-essential kernel-devel gcc gcc-c++ m4 ncurses-devel openssl openssl-devel unixODBC unixODBC-devel
# 2.安装erlang
sudo rpm -ivh erlang-21.3.8.18-1.el7.x86_64.rpm
# 3.安装socat
sudo rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm
# 4.安装rabbitmq-server
sudo rpm -ivh rabbitmq-server-3.7.27-1.el7.noarch.rpm
# 5.安装插件
sudo rabbitmq-plugins enable rabbitmq_management
# 6.修改配置
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.7.27/ebin/rabbit.app
改成{env,[
	...,
	{loopback_users,[guest]},
	...,
]}即可
# 7.启动
service rabbitmq-server start  
# 8.登录
192.168.154.131:15672 》》》id:guest mm:guest
```



# 工作流程解析

![image-20220720141006652](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220720141006652.png)

+ **Connection**：生产者和消费者之间的TCP连接。**Channel**：是一种轻量级的Connection，类比进程与线程。 

+ **Broker：**接收和分发消息的应用，即RabbitMQ Server。**交换机：**接收来自生产者的消息，并将其推送到队列中，必须明确交换机处理消息的具体模式。**队列：**RabbitMQ用于存储消息的数据结构，本质上是一个大的消息缓冲区。

+ **Virtual Host：**虚拟主机。如果将Queue比作数据表，那么虚拟主机就是一个数据库。每一个虚拟主机都是一个独立的逻辑分区。

**结论：**

+ RabbitMQ的连接是直接建立在Channel上的。
+ 消息需要通过交换机来选择发送到指定的队列。

# 使用

## Java操作流程

1. 添加依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-amqp</artifactId>
   </dependency>
   ```

2. 设置配置

   ```YML
   spring:
       rabbitmq:
           host: 127.0.0.1
           port: 5672
           username: guest
           password: guest
   ```

### 生产者

1. 创建并配置连接工厂`new ConnectionFactory();`
2. 由工厂创建连接`factory.newConnection();`
3. 通过连接创建信道`connection.createChannel();`
4. 由信道创建生产队列`channel.queueDeclare();`参数列表如下：
   1. 队列名称
   2. 消息是否持久化
   3. 消息是否共享
   4. 是否自动删除队列，【当队列中的消费者断连干净时，触发删除队列】
   5. 其他参数
5. 信道发送消息`channel.basicPublish();` 参数列表如下：
   1. 交换机
   2. 路由键，或者队列名称
   3. 其他参数
   4. 消息体

### 消费者

1. 创建并配置连接工厂`new ConnectionFactory();`

2. 由工厂创建连接`factory.newConnection();`

3. 通过连接创建信道`connection.createChannel();`

4. 信道监听生产队列，消费消息`channel.basicConsume();`，参数列表如下：

   1. 消费者监听的生产队列
   2. 消费成功后是否自动应当
   3. 消费成功的回调
   4. 消费失败的回调

   + 使用basicConsume方法接收：

     ```java
     //需要先声明两个回调函数
     // 消费成功的回调
     DeliverCallback deliverCallback=(consumerTag,delivery)->{
     	String message= new String(delivery.getBody());
     	System.out.println(message);
     };
     
     // 消费失败的回调
     CancelCallback cancelCallback=(consumerTag)->{
     	System.out.println(" 消息消费被中断");
     };
     	//消费生产队列的消息
         channel.basicConsume("消息队列", true, deliverCallback,cancelCallback);
     ```

   + 创建消费者，重写`handleDelivery();`

     ```java
      DefaultConsumer consumer = new DefaultConsumer(channel){
           @Override
           public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
             System.out.println("body ==="+new String(body));
           }
         };
         //消费生产队列的消息
         channel.basicConsume("消息队列", true, consumer);
       }
     ```



# 消息安全

## 消息应答

队列里的空间是有限的，需要及时清理已经被消费过的消息。为此RabbitMQ提供了应答机制，该机制在消息被**消费后**会将队列中对应的消息标记为已消费状态，供服务器及时清理。具体有两种方案：

### 自动应答

消费者一旦接收到消息，就会反馈，服务器立即对其清理。该机制比较危险，但后台服务出现bug时，消息会出现大量的丢失清空，极其不安全。

### 手动应答

```
Channel.basicAck(deliverCallback,boolean)//肯定确认(是否批量，不建议)
Channel.basicNack(*)//否定确认
Channel.basicReject()//否定确认
```

仅在程序员手动发送应答指令，服务器才会接收到该消息的具体状态，可以保证业务代码的逻辑合理，消息不丢失。业务当服务器未接受到ACK确认时，队列会一直保留这个消息，知道该消息被消费者消费成功，接收到ACK应答时，才会清理该消息。



## 持久化

为了保证RabbitMQ在退出或者异常情况下数据不丢失，需要将queue，exchange和Message都持久化。

### 交换机持久化

如果不设置exchange的持久化对消息的可靠性来说没有什么影响。但是当broker服务重启之后，exchange将丢失，就无法按照先前的设定正常地发送消息。`channel.exchangeDeclare(true);`

### 队列持久化

防止在RabbitMQ重启后，队列无了。`channel.queueDeclare();`，若是中途更改，则需要先将该队列删除。

### 消息持久化

`channel.basicPublish(MessageProperties.PERSISTENT_TEXT_PLAIN);`



## 发布确认

当生产者将消息发布到队列中，经IO保留在磁盘之后，RabbitMQ会向生产者一个反馈，该机制即为发布确认。保证消息绝对不丢失。有三种模式：

### 逐条确认（同步）

生产者发一条，服务器验一条，然后生产者后拿反馈后，才能继续发。

### 批量确认（同步）

生产者发若干条，服务器验一次，然后生产者拿到反馈后，才能继续发。

### 异步确认

当生产者将消息发送到信道中时，对消息进行编号，然后就一直发。好比快递，有了单号好维护，不必当面签收。之后由服务器对消息进行查验，无论持久化是否成功，都会异步地给生产者反馈。



# 工作模式

## Hello World

简单模式下队列与消费者呈比为**1 : 1**，当大量消息涌入队列时，极有可能会因为消费缓慢造成消息堆积，影响性能。

![image-20220722021827138](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220722021827138.png)

## Work Queue

让多个消费者同时监听消费同一个队列，同时采用轮询的方式保证消息被唯一消费。

![image-20220722023111542](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220722023111542.png)

## 发布/订阅模式 Publish/Subscribe

由于队列中的消息只能被消费一次，所以当我们需要将同一个消息发送到不同消费者时，以上的模式就无法满足需求。

![image-20220722153012905](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220722153012905.png)

该模式中生产者Channel直接关联交换机，默认不适用路由(使用默认路由)，做到多发。**<u>注意凡是RabbitMQ的组织结构中出现了交换机，就一定要交代routingKey，原因有二，1.交换机与队列的绑定少不了routingKey。2.生产者发消息时，没有routingKey或者写 `""` 导致 无法与后续routingKey匹配，将会导致消息全部被过滤掉。</u>**

## 路由模式

**routingKey：**作用的直接体现就是通过匹配结果，定制消息的走向。它的直接验证匹配环节是在交换机和队列的绑定处，但若是出现了多层的绑定，如死信架构最简单的两次绑定，就要仔细考虑生产者发送消息的 routingKey 了，避免消息在到达队列之前被阻死，造成消息丢失。

![image-20220722155523371](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220722155523371.png)

```
channel.exchangeDeclare(EXCHANGE, BuiltinExchangeType.DIRECT);
// 直连交换机，只向BindingKey和RoutingKey完全匹配的队列中发消息。

channel.exchangeDeclare(EXCHANGE, BuiltinExchangeType.FANOUT);
// 扇形交换机，向绑定自身的所有队列发消息。

channel.exchangeDeclare(EXCHANGE, BuiltinExchangeType.TOPIC);
// 主题交换机，只向BindingKey和RoutingKey完全匹配的队列中发消息。#表示一个或多个字符，*表示一个字符。

channel.exchangeDeclare(EXCHANGE, BuiltinExchangeType.HEADERS);
// 头交换机，不处理路由,而是根据发送的消息内容中的headers属性进行匹配。
```

## Topic（通配符模式）

是路由模式的改良版，在通配符的加持下，能与通配字符串路由键进行匹配绑定。



# 幂等性

消息被重复消费了。造成了业务上的错误，或者不合理。

**解决思路：**

一般使用全局ID、唯一标识比如时间戳或者UUID、消费者消费MQ中的消息、MQ的该id来判断，每次消费消息时用该id先判断该消息是否已消费过。

**手段：**

+ 唯一ID + 指纹码机制：通过唯一标识查询数据库，进行比对判断
+ **Redis原子性**：setnx命令，天然具有幂等性，而且元素不重复。【推荐】



# 优先级队列

1. 配置队列优先级

   ```java
   	@Bean("queue1")
   	public Queue queue1() {
   		HashMap<String, Object> map = new HashMap<>();
   		map.put("x-max-priority", 10);//配置0-10的优先级，越大越优先
   		return QueueBuilder.durable(NORMAL_QUEUE_1).withArguments(map).build();
   	}
   ```

2. 设置消息优先级

   ```java
   	@Test
   	public void send() throws InterruptedException {
   		for (int i = 1; i <= 10; i++) {
               if(i==3){
                   //设置优先级
              		new AMQP.BasicProperties().builder().priority(5).build();
   				workProducer.send(); 
               }
   		}
   	}
   ```

   

# 日志与监控

**日志位置：** `/var/log/rabbitmq/rabbit@xxx.log`