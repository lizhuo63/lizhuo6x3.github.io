# 简介

+ **特点：**
  + 基于AMQP(高级消息队列协议)实现的，可复用的消息系统
+ **优点：**
  + 性能好，高并发
  + 吞吐量达万级，时效性微秒级
  + 健壮，稳定
+ **缺点：**商业版需要收费,学习成本较高
+ **适用场景：**数据量没有那么大，中小型公司优先选择功能比较完备的 RabbitMQ。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-10_06-15-11.png)

+ **生产者**
  产生数据发送消息的程序是生产者

+ **Connection**

  生产者和消费者之间的TCP连接

  + **Channel**

    建立 TCP Connection 的开销很大，Channel是一种轻量级的Connection 

+ **Broker：**接收和分发消息的应用，即RabbitMQ Server

  + **交换机**

    接收来自生产者的消息，并将其推送到队列中。必须明确交换机处理消息的具体模式

  + **队列**

    RabbitMQ用于存储消息的数据结构，本质上是一个大的消息缓冲区。

+ **消费者**
  是一个等待接收消息的程序。

​	

# 使用

## 编写流程

### 生产者

1. 创建并配置连接工厂`new ConnectionFactory();`
2. 由工厂创建连接`factory.newConnection();`
3. 通过连接创建信道`connection.createChannel();`
4. 由信道创建生产队列`channel.queueDeclare();`
5. 信道发送消息`channel.basicPublish();`

### 消费者

1. 创建并配置连接工厂`new ConnectionFactory();`

2. 由工厂创建连接`factory.newConnection();`

3. 通过连接创建信道`connection.createChannel();`

4. 信道监听生产队列，消费消息

   + `channel.queueDeclare();`

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

## springboot环境

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

3. 

## 基本模式