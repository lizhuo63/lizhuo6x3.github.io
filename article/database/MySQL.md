# 官方文档8

[官方文档](https://git-scm.com/book/zh/v2)

# 原理

## 数据存储

### 1.磁盘的存取原理

为了实现数据的持久化存储，mysql将数据存储到了磁盘上。为了高效查询数据，mysql将磁盘上的相关数据加载到内存中（走磁盘IO），并将查询出的结果数据返回给客户端。

操作系统将磁盘上的数据文件加载到内存中（是以块为最小单位的）。mysql5.5之后，默认以InnoDB作为存储引擎，InnoDB是以数据页来存储数据、索引的(大小默认为16KB)，操作系统将数据库中的数据加载到内存也是以数据页为基本单位。



## 架构

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-29_21-22-00.png)

### 执行顺序

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-29_21-43-13.png" style="zoom:67%;" />

#### NO.1连接器

第一步，连接数据库。连接器负责跟客户端建立连接、获取权限、维持和管理连接。连接命令一般是这么写的：`mysql -h$ip -P$port -u$user -p`。

1. 在完成经典的TCP握手后，连接器就要开始认证你的身份，这个时候用的就是你输入的用户名和密码。
2. 认证通过后，连接器会到权限表里面查出你拥有的权限。之后，这个连接里面的权限判断逻辑都将依赖于此时读到的权限。

3. 连接完成后，如果你没有后续的动作，这个连接就处于空闲状态，客户端如果太长时间没动静，连接器就会自动将它断开。这个时间是由参数wait_timeout控制的，默认值是8小时。
4. 如果在连接被断开之后，客户端再次发送请求的话，就会收到一个错误提醒： Lost connection to MySQL server during query。这时候如果你要继续，就需要重连，然后再执行请求了。

#### NO.2查询缓存

连接建立完成后，执行逻辑就会来到第二步：查询缓存。

会频繁失效：**【只要有对一个表的更新，这个表上所有的查询缓存都会被清空】**，可以将 query_cache_type 设置为 DEMAIND，这样默认将不使用查询缓存，Mysql8已将查询缓存废除。

1. MySQL拿到查询请求后，会先到查询缓存看看，如果命中就直接返回，否则进入后续执行阶段。
2. 可以将参数query_cache_type设置成DEMAND，这样对于默认的SQL语句都不使用查询缓存。

#### NO.3分析器

如果没有命中查询缓存，就要开始真正执行语句。首先，MySQL需要知道你要做什么，因此需要对SQL语句做解析。

1. 分析器先会做“词法分析”。MySQL需要识别出里面的字符串分别是什么，代表什么。
2. 随后做“语法分析”。语法分析器会根据语法规则，判断输入的SQL语句是否满足MySQL语法。

#### NO.4优化器

经过了分析器，MySQL就知道你要做什么了。在开始执行之前，还要先经过优化器的处理。

1. 在表里面有多个索引的时候，决定使用哪个索引
2. 在一个语句有多表关联（join）的时候，决定各个表的连接顺序

比如有两种执行方法的逻辑结果是一样的，但是执行的效率会有不同，而**优化器的作用就是决定选择使用哪一个方案。**

#### NO.5执行器

通过优化器知道了该怎么做，于是就进入了执行器阶段，开始执行语句。

1. 开始执行的时候，要先判断一下你对这个表有没有执行查询的权限，如果没有，就会报错。
2. 打开表的时候，执行器就会根据表的引擎定义，去使用这个引擎提供的接口。执行器相对于一个门面，而引擎才是实际的处理器。

#### NO.6存储引擎层

存储引擎层负责数据的存储和提取。不同的存储引擎共用一个**Server层**，也就是从连接器到执行器的部分。

#### ==7后续数据更新

+ 更新数据它更新的是缓存中的数据，为防止内存数据丢失，会有后台进程对缓存更新进行监听并实时刷新**redo.log**（追加日志），然后由LSN机制对比磁盘和缓存数据，找到差异点并实现持久化。

+ 正常退出时，mysql会将缓存池中的数据持久化下来，保证下次启动时缓冲池中的热点数据不会丢失。意外宕机时，可根据redo.log恢复数据。

+ 最后三步将redo log 和binlog的写入纳入一个事务中，可以保证两个日志的记录效果一致，利于数据恢复。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-09_15-51-06.png)

### 存储引擎

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-29_21-50-22.png)

#### **InnoDB存储引擎**

+ InnoDB主要特性有：支持事务、行级锁和外键。

+ InnoDB是为处理巨大数据而设计的，它的CPU效率是其他引擎锁不能匹敌的。

+ InnoDB将表和索引维护在一个逻辑表空间中，表空间可以包含数个文件（或原始磁盘文件）。InnoDB表可以是任何尺寸，即使在文件尺寸被限制为2GB的操作系统上

+ InnoDB存储表中的数据时，每张表的存储都按主键顺序存放，如果没有指定主键，InnoDB会为每一行生成一个6字节的ROWID，并以此作为主键

使用InnoDB时，MySQL将在MySQL数据目录下创建一个名为ibdata1的10MB大小的自动扩展数据文件，以及两个名为ib_logfile0和ib_logfile1的5MB大小的日志文件。

场景：对事务的完整性要求比较高（比如银行），要求实现并发控制（比如售票），那选择InnoDB有很大的优势。如果需要频繁的更新、删除操作的数据库，也可以选择InnoDB，因为支持事务的提交（commit）和回滚（rollback）。

#### **MyISAM存储引擎**

MyISAM基于ISAM存储引擎，它拥有较高的插入、查询速度，但不支持事物和外键。

MyISAM主要特性有：

+ 每个MyISAM表最大索引数是64，每个索引最大的列数是16。

+ 最大的键长是1000字节

+ BLOB和TEXT列可以被索引，支持FULLTEXT类型的索引，而InnoDB不支持这种类型的索引

+ NULL被允许在索引的列中，这个值占每个键的0~1个字节

+ 每个MyISAM类型的表都有一个AUTO_INCREMENT的内部列，因而MyISAM类型表的AUTO_INCREMENT列更新比InnoDB类型的AUTO_INCREMENT更快

+ 可以把数据文件和索引文件放在不同目录

+ VARCHAR和CHAR列可以多达64KB

存储格式：

1、静态表（默认）：字段都是定长的。存储非常迅速、容易缓存，出现故障容易恢复；占用空间通常比动态表多。

2、动态表：占用的空间相对较少，但是频繁的更新删除记录会产生碎片，需要定期执行optimize table或myisamchk -r命令来改善性能，而且出现故障的时候恢复比较困难。

3、压缩表：使用myisampack工具创建，占用非常小的磁盘空间。因为每个记录是被单独压缩的，所以只有非常小的访问开支。

使用MyISAM引擎创建数据库，将产生3个文件。文件的名字以表名字开始，扩展名之处文件类型：frm文件存储表定义、数据文件的扩展名为.MYD（MYData）、索引文件的扩展名时.MYI（MYIndex）。

场景：如果表主要是用于插入新记录和读出记录，那么选择MyISAM能实现处理高效率。

## 日志

### redo log

 redo log 是InnoDB 引擎特有的日志。在对数据进行变更时，mydql会先清空所涉及表的缓存，经过分析和优化之后，由执行器具体执行数据更改。为了高效地持久化数据，mysql会将变更记录存在redo log中。【每个redo log的大小固定为1GB，为从头到尾循环覆盖写的模式】。自此就可依据redo log解决异常宕机后的数据持久化，即 **crash-safe**。

###  binlog

binlog是server层自己的日志实现，因为早期还没有InnoDB 引擎，默认的 MyISAM 没有crash-safe 能力。binlog 日志只能用于归档。

### 两种日志的区别：

1. redo log 是 InnoDB 引擎特有的； binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。
2. redo log 是物理日志，记录的是 “ 在某个数据页上做了什么修改 ” ； binlog 是逻辑日志，记录的
是这个语句的原始逻辑，比如 “ 给 ID=2 这一行的 c 字段加 1 ” 。
3. redo log 是循环覆盖写的，空间固定； binlog 是可追加写，到一定大小后会切换写入下一个文件。 

# 索引

 

索引是一种特殊的数据结构，用于快速查找数据库表中的特定记录，

1. 将数据加载到内存中去
2. 从内存中二分查找，定位数据返回

+ **单值查询：**类似 `select name where id=1`, 由索引key定位数据value

  + 哈希表：O(1)  将索引列存入桶数组（充当目录），数据挂在桶下

+ **范围查询：**`select name where id > 5 and id < 12`，为了高效，希望索引列是有序的。

  + **有序数组：**O(1)，不利于数据增删

  + 搜索二叉树：O(logN)，存在数据倾斜（树不平衡）的情况

  + **AVL树：**解决了二叉树的不平衡问题，结构调整频繁引发高IO，效率差

  + **红黑树：**降低了AVL树的结构调整的要求，性能较AVL树更优，但仍然不适用充当磁盘介质（IO代价高）

  + 跳表：

  + **B tree：**

    + 节点中同时存储索引数值和真实数据，命中后可直接返回结束
    + 加大了存储占用，由于页的大小有限，会加大树高，降低性能

    ​	![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-09_13-01-03.png)

    

  + **B+tree：**

    + 将索引数值和真实数据分离，导致了可能会出现重复命中的情况，但是效率上影响不大

    ![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-09_13-08-47.png)

​			

### 索引失效情况

+ **索引列非独立：** sql语句中的索引列必须是独立出现的，`where id + 1 = 5`就不行。
+ **索引列不完整：**主要出现在联合索引中
+ **多条件范围查询：**无法同时使用多个索引
+ **联合索引：**只对第一个有效
+ **or：**两边都要有索引











# 优化

根据MYSQL执行顺序，可依次从连接、解析和执行三个维度进行优化。

+ 从对整个系统的影响来看，一个频繁执行的高并发查询远比一个低并发查询要危险。

## 连接层

### 服务端优化

+ 添加可用的连接数，修改环境变量`max_connections`（默认151）
+ 及时释放空闲连接，修改环境变量 `wait_timeout`（默认28800 [8小时]）

###  客户端优化

+ 使用数据库连接池如：Druid

## 解析层

### 慢日志

开启慢查询日志是有性能代价的，因此MySQL默认是关闭的。

1. 查看当前慢查询状态： `show variables like 'slow_query%';`

2. 更改慢查询阈值：`show variables like '%long_query%';`，默认10s

    ```sql
    # 开启慢查询日志
    slow_query_log=ON
    long_query_time=2
    slow_query_log_file=/var/lib/mysql/slow.log
    
    # 动态修改
    mysql> set @@global.slow_query_log=1;
    mysql> set @@global.long_query_time=2;
    ```

3. 查看慢查询日志：`cat /var/lib/mysql/695f5026f0f6-slow.log` 

4. 慢日志查询的工具`mysqldumpslow`

    + 使用

        ```
        mysqldumpslow [ OPTS... ] [ LOGS... ] # 使用格式
        
        -s ORDER 排序依据 (al, at, ar, c, l, r, t)，'at' 是默认值
        	al：平均锁定时间
        	ar：发送的平均行数
        	at：平均查询时间
        	c：计数
        	l：锁定时间
        	r：发送的行
        	t：查询时间
        -r 反转排序顺序（最大的最后而不是第一个）
        -t n 只显示前n个查询
        -a 不要将所有数字抽象为N并将字符串抽象为'S'
        -n n个抽象数字，名称中至少有n个数字
        -g PATTERN grep：仅考虑包含此字符串的 stmts
        -h HOSTNAME 用于 *-slow.log 文件名的数据库服务器主机名（可以是通配符），
        默认为'*'，即匹配所有
        -i NAME 服务器实例的名称（如果使用 mysql.server 启动脚本）
        -l 不要从总时间中减去锁定时间
        
        #  ===示例===
        # 访问次数最多的10条SQL：
        mysqldumpslow -s r -t  10 /var/lib/mysql/695f5026f0f6-slow.log
        得到按照时间排序的前10条里面含有左连接的SQL：
        mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/695f5026f0f6-slow.log
        ```

    + 将未使用索引的SQL录入慢查询日志

        ```sql
        show variables like 'log_queries_not_using_indexes'; # 查看设置，默认关闭
        set global log_queries_not_using_indexes = on; # 设置
        ```

### 线程清理

+ 运行`show full processlist`查看MySQL中运行的所有线程，将有问题的直接通过id来kill。

    **Id**：线程的唯一标志

    **User**：启动该线程的用户，普通用户只能查看自己的线程

    **Host**：哪个ip和端口发起的连接

    **db**：线程操作的数据库

    **Command**：线程的命令

    **Time**：操作持续时间

    **State**：线程的状态

    **Info**：SQL语句的前100个字符

# SQL优化

## DQL执行顺序

```sql
SELECT(9) DISTINCT column,… # (8) 选择字段 、(9)去重
AGG_FUNC(column or expression),… # (6) 聚合函数
FROM [left_table] # (1) 选择表
<join_type> JOIN <right_table> # (3) 链接
ON <join_condition> # (2) 链接条件
WHERE <where_condition> # (4) 条件过滤
GROUP BY <group_by_list> # (5) 分组
HAVING <having_condition> # (7) 分组过滤
ORDER BY <order_by_list> # (10) 排序
LIMIT count OFFSET count; # (11) 分页
```

1. 减少数据访问：设置合理的字段类型，启用压缩，通过索引访问等**减少磁盘IO**
2. 返回更少的数据：只返回需要的字段和数据分页处理 **减少磁盘io及网络io**
3. 减少交互次数：批量DML操作，函数存储等减少数据**连接次数**
4. 减少服务器CPU开销：尽量减少数据库排序操作以及全表查询，减少cpu内存占用
5. 利用更多资源：使用表分区，可以增加并行操作，更大限度利用cpu资源

总结就三点:

- 最大化利用索引；
- 尽可能避免全表扫描；
- 减少无效数据的查询；

+ 
