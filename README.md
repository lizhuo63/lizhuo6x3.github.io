<link rel="stylesheet" href="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/docsify-note/css/local.css" type="text/css">

![hello1_show](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/docsify-note/media/img/hello1.png ':class=hello1_show')
![hello2_show](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/docsify-note/media/img/hello2.png ':class=hello2_show')

> [!TIP]
> 本栏主要是收纳个人创造、学习总结的文章。力在记录学习历程、偶时感悟，便于日后拾遗、复盘和使用。
> **本栏的核心方向**：
>
> + 基础理论、环境搭载记录、java后端、重构经验、常用前端、项目总结。

# 🌏 环境搭建
> + [docker手册](article/common_tools/environment/docker.md)
> + [node安装](article/common_tools/environment/node.md)
> + [git使用](article/common_tools/environment/git.md)

# ☕ java
<!-- tabs:start -->
#### **异常**
> + [异常](article/required_skills/java/java_exception.md)
#### **注解**
> + [注解](article/required_skills/java/java_annotation.md)
#### **反射&代理**
> + [反射](article/required_skills/java/java_reflex.md)
> + [静态代理]()
> + [动态代理]()
#### **IO**

#### **集合**
+ [Collection-单列集合](article/required_skills/java/java_collection.md?id=collection) 
  + [List-存取有序，元素可重复、有索引，支持null](article/required_skills/java/java_collection.md?id=list)
    + Vector[底层数据结构是数组。线程安全]
    + ArrayList[底层数据结构是数组。线程不安全]
    + LinkedList[底层数据结构是链表。线程不安全]
  + [Set-存取无序、唯一，支持null](article/required_skills/java/java_collection.md?id=set)
    + HashSet[底层数据结构是哈希表(是一个元素为链表的数组) + 红黑树]
    + TreeSet[底层数据结构是红黑树(是一个自平衡的二叉树)]
    + LinkedHashSet[底层数据结构由哈希表(是一个元素为链表的数组)和双向链表组成。]
+ [Map-双列集合](article/required_skills/java/java_collection.md?id=map)
  + HashMap
  + TreeMap
  + HashTable
  + LinkedHashMap
  + ConcurrentHashMap
#### **函数式**

#### **多线程**
> + [java线程](article/required_skills/java/multi_thread/thread.md)

#### **JVM**
> + [class结构](article/required_skills/java/jvm/jvm_class_structure.md)
> + [JVM数据模型](article/required_skills/java/jvm/jvm_data_model.md)
> + [JVM内存模型](article/required_skills/java/jvm/jvm_memory_model.md)
> + [类加载机制](article/required_skills/java/jvm/jvm_class_loading.md)
> + [对象创建](article/required_skills/java/jvm/jvm_object_creating.md)
> + [GC](article/required_skills/java/jvm/jvm_GC.md)

#### **源码**

<!-- tabs:end -->

# 🛢️ 数据库
<!-- tabs:start -->
#### **MySQL**
> 实用篇
  

> 原理篇
  + [MySQL的架构与执行](article/required_skills/database/mysql/mysql_framework_execution.md)
  + [MySQL的索引](article/required_skills/database/mysql/mysql_indexes.md)
  + [MySQL的事务和锁](article/required_skills/database/mysql/mysql_transaction.md)
  + [MySQL优化](article/required_skills/database/mysql/mysql_optimization.md)
  + [MySQL](article/required_skills/database/MySQL.md)
#### **Redis**
> + [Redis](article/required_skills/database/redis/Redis.md)
<!-- tabs:end -->

# 🛴 框架
<!-- tabs:start -->
#### **Server[服务器]**

#### **View**

#### **Spring**
> + [Spring](article/required_skills/frame/spring/spring.md)
> + [SpringMVC](article/required_skills/frame/spring/springMVC.md)
> + [SpringBoot](article/required_skills/frame/spring/springBoot.md)

#### **ORM**
## Mybatis
> + [Mybatis执行流程与架构](article/required_skills/frame/orm/mybatis/mybatis_frame.md)

#### **RPC**

#### **MQ**

Message Queue 消息队列，一种用于上下游可跨线程的通信机制。核心功能包括：流量消峰、应用解耦、异步通信(处理)。目前较热门的MQ有RabbitMQ，RocketMQ，Kafka。

**Kafka：**专为为大数据而生，有百万级TPS的吞吐量，kafa是分布式的，数据安全性高，消息有序。但是消费失败不支持重试，而且单机宕机后会消息乱序，当单机队列超过64个时，CPU会飙高，响应变慢。

**RocketMQ：**广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理，binlog分发等场 景。 单机吞吐量十万级，可用性非常高，分布式架构，数据安全，支持10亿级别的消息堆积且不会影响性能。但仅仅对Java支持完善。

**RabbitMQ：** 并发性能较好，实效性为微秒级；吞吐量到万级，MQ功能比较完备，健壮、稳定、易用、跨平台。

#### **Search**

<!-- tabs:end -->


# 💻 计算机理论基础
<!-- tabs:start -->
#### **操作系统**
> + [进程管理](article/basic_knowledge/computer/os/process_management/process.md)
> + [内存管理]()
> + [磁盘管理]()
#### **🕸️ 计算机网络**

#### **🌐 协议**

<!-- tabs:end -->



# 基础技能

## 🎰 数据结构
<!-- tabs:start -->
#### **线性表**
+ 链表
+ 列表

#### **树**

<!-- tabs:end -->
## 🧮 算法
<!-- tabs:start -->
#### **排序**
+ [排序](article/basic_knowledge/algorithm/sort.md)

#### **查找**
+ [查找](article/basic_knowledge/algorithm/search.md)
<!-- tabs:end -->
## 🧱 重构和设计模式
<!-- tabs:start -->
#### **UML类图**

#### **创建型**
+ [简单工厂（几乎不用）](article/required_skills/rebuild/creational/factory.md?id=简单工厂)
+ [工厂方法](article/required_skills/rebuild/creational/factory.md?id=工厂方法)
+ [抽象工厂（拼装）](article/required_skills/rebuild/creational/factory.md?id=抽象工厂)
+ [建造者](article/required_skills/rebuild/creational/builder.md)

<!-- tabs:end -->

# ✍️学习总结

<!-- tabs:start -->
#### **1**
> + [我眼中的JavaWeb](article/learning_summary/java_web.md)
<!-- tabs:end -->
