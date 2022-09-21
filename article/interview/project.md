# 线上bug

## 排查

线上故障主要会包括 CPU、磁盘、内存以及网络问题

总体目标：**尽快恢复服务，消除影响**。解决思路：

+ **cpu飙高**，提示服务器负载异常
  - 频繁 gc
    - **arthas**，查看CPU占用情况
      - 飙高的是GC线程
      - 飙高的是业务线程
  - 死循环、线程阻塞、io wait...etc
+ 



# 并发

### 高并发下抢单、限时秒杀怎么处理

使用redis队列实现，比如有20w人涌入下单页面进行秒杀，而货存只有500。首先把请求下单的用户放入redis队列中，然后再转到支付页面。当队列中的用户达到500时，后续用户就转到秒杀结束页面。为了抗住高并发下的压力，可使用到**页面静态化**技术和**秒杀按钮**过滤掉无效请求。另外可使用更高效的数据库比如redis，同时对秒杀数据做一个预热。

### 线程池



### 怎么保证线程变量是私有不共享



### ThreadLocal



# 安全

### 如何保证幂等性

**前端拦截**

具体表现为通过按钮来阻止用户在接口调用后的的重复点击。

**后端处理**

1. 为幂等性接口的请求设置唯一的ID
2. 成功处理该接口后，将此ID存入redis中
3. 在请求该幂等性接口之前进行逻辑判断处理

**全局ID**

后端接收由前端传入的一个全局ID，在新增数据时就以该全局ID为主键插入到数据库，同时要在目标数据库表中为此全局ID设置唯一约束。

# Web

### get与post区别







# Redis

### redis令牌桶

令牌桶算法，可以应对短暂的突发流量，对于流量不均匀的情况特别有用，不会频繁的触发限流，对调用方比较友好。

### 缓存击穿



### 布隆过滤器



### set和zset的区别，项目使用



### 持久化机制



### 如何设置过期时间



# rabbitmq

### rabbitmq如何使用



# ES

### es如何使用



# 业务问题

## 解决大批量的用户注册

方案一：结合验证码，在注册接口内设置一个与注册业务和用户IP相关联的redis_Key并设置过期时间。





# Mysql

### 存储引擎



### 存储上限，索引类型



## mysql行列转换

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220915145001662.png" alt="image-20220915145001662" style="zoom: 80%;" />

**列转行：**

![image-20220915150928500](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220915150928500.png)

**控制函数（case...when...then） + 聚合函数（max/sum）+ 分组（group by）**

```sql
-- sum和max效果一样
select
	userid,
	sum(case when subject='语文' then score else 0 end) as '语文',
	sum(case when subject='数学' then score else 0 end) as '数学',
	sum(case when subject='英语' then score else 0 end) as '英语',
	sum(case when subject='政治' then score else 0 end) as '政治'
from 
	tb_score
group by 
	userid
```

**控制函数（if） + 聚合函数（max/sum）+ 分组（group by）**

```sql
select
	userid,
	sum(if(subject = '语文', score, 0)) as '语文',
	sum(if(subject = '数学', score, 0)) as '数学',
	sum(if(subject = '英语', score, 0)) as '英语',
	sum(if(subject = '政治', score, 0)) as '政治' 
from 
	tb_score 
group by
	userid
```



**行转列：**

**group by + case when**