# String

Redis 的字符串又称简单动态字符串（Simple Dynamic String），简称SDS。它是动态的，类似 Java 的ArrayList。它采用预分配冗余空间的方式来减少内存的频繁分配。当字符串长度小于 1M 时，均是加倍扩容，如果超过 1M，扩容时一次只会多扩 1M 的空间。字符串最大长度为 512M。

![image-20220713200733557](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220713200733557.png)

## 操作

数据的存取都是逐条进行的，单数据操作与多数据操作的区别在于指令传输的区别，单：每次发送一条指令，多：打包发送一次完成。所以当指令量过大时一定要用多指令操作来提高效率。

 **查**

+ 值的长度：`strlen k`
+ 单个：`get k`
+ 多个：`mget k1 k1 k3`
+ 查值的指定范围[m,n]内容：`getrange s1  m n`

**增**

+ 单个：`set k v`
+ 批量：`mset k1 v1 k2 v2 k3 v3`
+ 判断：`setnx k v`，仅当键不存在的时候才执行
+ 判断：`msetnx k1 v1 k2 v2 k3 v3`，**要求所有键都不存在**

**删**

+ `del k`
+ `del k1 k2`

**改**

+ 全覆盖：`set k v`，`getset k v`
+ 追加：`append k v`，k不存在就新建
+ 范围覆盖：`setrange k index v`，从指定位置进行覆盖。若index>length，会以空符/x00填充。
+ 针对整数：自增x：`incrby k x`；自减x：`decrby k x`(可省略1和by)，**超出java最大Long值会报错。**
+ 针对浮点数：加：`incrbyfloat s1 2.1`；减：`decrbyfloat s1 2.1`。指令值必须为浮点数，默认小数位为17

**过期**

+ `setex k 有效秒数 v`
+ `expire k 有效秒数`

### 总结

**nx操作具有事务性质**。**1.** 在对Java对象进行操作时因该尽量使用它，防止异常引发的局部插入导致数据残缺。**2.** 针对数值进行计数器设置可实现服务端的功能限制，例如登录试错次数，网站防爬等等。



# List

Redis 的 List相当于 Java 的 LinkedList，它是链表而不是数组。这意味着 List 的增删非常快。当列表弹出了最后一个元素之后，该数据结构自动被删除，内存被回收。Redis 的列表结构常用来做异步队列使用。也能充当栈。

## 操作

**删**

+ 左侧弹出：`lpop k`
+ 右侧弹出：`rpop k`
+ 删除n个v：`lrem k n v`，n>0【从左开始】；n<0【从右开始】

**查**

+ 元素个数：`llen k`
+ 从左到右获取List的[m,n]范围值：`lrange k m n`，0是左起第一个（后入），-1是右起第一个（先入）
+ 获取从左侧第 i 个入列的元素：`lindex k i`

**增**

+ 从左往右插：`lpush k v1 v2 v3` -> (v3,v2,v1)
+ 从右往左插：`rpush k v1 v2 v3` ->(v1,v2,v3)
+ 将value插到pivot的左或右边：`linsert key before|after pivot value`
+ 判断左插：`lpushx k v`
+ 判断右插：`rpushx k v`
+ 右弹左插：`rpoplpush k1 k2`，将k1右弹插入k2左侧

**改**

+ 改左起第 i 个`lset k i v`

# Hash



## 操作

**查**

+ `hget k f`；`hmget k f1 f2 f3`
+ 查看字段数：`hlen k`
+ 查看所有字段：`hkeys k`
+ 查看所有值：` hvals k`
+ 查看所有字段和值：`hgetall k`
+ 字段值的长度：`hstrlen k f`
+ 字段的存在性：`hexists k f`

**增**

+ ` hmset k f1 v1 f2 v2`，` hset k f1 v1 f2 v2`，若 k f 存在，就会新值覆盖
+ 判断f存在：` hsetnx k f v`，



**改**

+ 整数增减：`hincrby k f  n`，n可为正负。
+ 浮点数：`hincrbyfloat k f n`，n可为正负，**可为整数**

**删**

+ `hdel k f`



# Set

当Set 内没有元素时，Redis会会回收它的键。

## 操作

**查**

+ 获取所有元素：`smembers k`
+ 随机获取n个元素：`srandmember k n`，n>0不重复；n<0，可重复。n>size时，获取所有元素。
+ 获取元素数量：`scard k`
+ 判断元素存在性：`sismember  k v`

**增**

+ `sadd k v1 v2 v3`，重复元素自动忽略

**删**

+ `srem k v1 v2 v3`，删除元素
+ `spop k n`，随机获取弹出n个元素 

**改**

+ ` smove k1 k2 a`，将k1中的a移到k2

**交**

+ 获取交集元素：`sinter st1 st2`
+ 将st1、st2的交元素存入st3：`sinterstore st3 st1 st2`

**并**

+ 获取并集元素：`sunion st1 st2`
+ 将st1、st2的并元素存入st4：`sunionstore st4 st1 st2`

**差**

+ 获取差集元素：` sdiff st1 st2`
+ 将st1、st2的差元素存入st5：`sdiffstore st4 st1 st2`

## 总结

**1.** Set 自带重复过滤机制，可以应用到计数环境中，如点赞，投票。**2.** Set 的随机获取功能可支持抽奖场景应用。**3.** Set对集合的交并差功能支持社交资源共享应用。



# ZSet







# 数据时效

## 存在原因

在内存占用与CPU占用之间找一种平衡，让CPU在不忙的情况下删一些数据，保证内存有效的快速释放，进一步保证Redis的性能以及服务器的稳定【不宕机】

## 数据结构

TTL返回的值有三种情况：正数，-1，-2

- **正数**：代表该数据在内存中还能存活的时间
- **-1**：永久有效的数据
- **2** ：已经过期的数据 或被删除的数据 或 未定义的数据

![image-20220715093452278](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220715093452278.png)

过期数据是一块独立的存储空间，Hash结构，field是内存地址，value是过期时间，保存了所有key的过期描述，在最终进行过期处理的时候，对该空间的数据进行检测， 当时间到期之后通过field找到内存该地址处的数据，然后进行相关操作。

## 实现策略

**定时删除：**

创建一个定时器，当key达到过期时间时，由定时器任务立即执行对键的删除操作【到点就删】，是在用CPU性能换取存储空间（拿时间换空间）。

- **优点**：节约内存，到时就删除，快速释放掉不必要的内存占用
- **缺点**：CPU压力很大，无论CPU此时负载量多高，均占用CPU，会影响redis服务器响应时间和指令吞吐量

**惰性删除：**

数据到达过期时间时，不做处理。等下次访问该数据时再做判断。是在用存储空间换取CPU性能（拿空间换时间）

1. 如果未过期，返回数据
2. 发现已过期，删除，返回不存在

- **优点**：节约CPU性能，发现必须删除的时候才删除
- **缺点**：内存压力很大，出现长期占用内存的数据

**定期删除：**

Redis会对数据库进行轮询操作，每次轮询会依次遍历16个库并取样检查时效并删除。当当前库的删除量超过取样量W的1/4时，Redis会认定当前库的过期数据较多，会再次取样(循环)，直至核算标准低于1/4才会检查下一个库。Redis启动时会去读取server.hz的值(默认10)【`info server`】，意为每秒执行10次轮询操作。

+ W是ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP的配置值，可自行设置
+ 当一次检查【activeExpireCycle()】执行完毕没能遍历完所有的库是，下一次检测就会以上次终止的库为起点继续执行。

**特点：**

+ 可自定义检测频率，进而设定CPU占用峰值
+ 定期随机清理，能重点抽查过期量比较大的库



# 数据淘汰

与数据时效一样是为了内存利用而设计的，区别是数据淘汰是紧急情况，处理插入数据时内存不足的情况。

Redis在执行每一个命令前，会调用freeMemoryIfNeeded()检测内存是否充足。如果内存不满足新加入数据的最低存储要求，redis要临时删除一些数据为新数据腾地方。清理数据的策略称为逐出算法。

**注意：**逐出数据的过程不是100%能够清理出足够供使用的内存空间，如果不成功则反复执行。当对所有数据尝试完毕，如不能达到内存清理的要求，将报错OOM。

**相关配置：**

```
maxmemory ?mb   最大可使用的物理内存的比例，默认值为0[不限制],通常设置在50%以上
maxmemory-samples count   每次选取待删除数据的个数
maxmemory-policy policy   对数据进行删除的选择策略
```

**策略policy有3类8种**

```
**第一类**：检测易失数据（可能会过期的数据集server.db[i].expires ）
volatile-lru：挑选最近最久没用的数据淘汰
volatile-lfu：挑选最近使用次数最少的数据淘汰
volatile-ttl：挑选将要过期的数据淘汰
volatile-random：任意选择数据淘汰

**第二类**：检测全库数据（所有数据集server.db[i].dict ）
allkeys-lru：挑选最近最久没用的数据淘汰
allkeLyRs-lfu：：挑选最近使用次数最少的数据淘汰
allkeys-random：任意选择数据淘汰，相当于随机

**第三类**：放弃数据驱逐
no-enviction（驱逐）：禁止驱逐数据(redis4.0中默认策略)，会引发OOM(Out Of Memory)
```

