# Mysql

### 三大范式

### MyISAM与InnoDB区别

### MyISAM索引与InnoDB索引的区别

+ InnoDB索引是聚簇索引，MyISAM索引是非聚簇索引。
+ InnoDB只有索引中的叶子节点存储着行数据，因此主键索引非常高效。
+ MyISAM索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据。
+ InnoDB非主键索引的叶子节点存储的是主键和其他带索引的列数据，因此查询时做到覆盖索引会
  非常高效

### *索引有哪几种类型

### *SQL约束有哪几种

### *事务四大特性



### 百万级别或以上的数据如何删除

1. 可以先删除索引（此时大概耗时三分多钟）

2. 然后删除其中无用数据（此过程需要不到两分钟）
3. 删除完成后重新创建索引(此时数据较少了)创建索引也非常快，约十分
    钟左右。

  **原因：**索引需要额外的维护成本，对数据的增加,修改,删除都会产生对索引文件额外的操作消耗额外的IO，会降低增/改/删的执行效率。

### 非聚簇索引一定会回表查询吗

不一定，若涉及到的查询字段全部命中了索引，就不会进行回表查询，而是通过覆盖索引的方式拿数据。

### 为什么需要注意联合索引中的顺序

因为联合索引的命中规则是按照最初该联合索引声明时的字段顺序，从左到右依次、逐个命中，如果给出的条件顺序不符，将会导致不符字段以及之后的字段索引失效，影响查找性能。

### *事务的隔离级别

### 行锁如何实现

在sql语句后加上for update即可，但是需要命中索引，否则行锁会升级为表锁。

### 数据库的乐观锁和悲观锁

**悲观锁**：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。在查询完数据的时候就把事务锁起来，直到提交事务。实现方式：使用数据库中的锁机制

**乐观锁**：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。在修改数据的时候把事务锁起来，通过version的方式来进行锁定。实现方式：一般会使用版本号机制或CAS算法实现。

### in和exists区别

mysql中的in语句是把外表和内表作hash 连接，而exists语句是对外表作loop循环，每次loop循环再对内表进行查询。

### varchar(50)中50的涵义

在内存中最多存放50个字符，varchar(50)和(200)存储hello所占物理存储空间一样，但后者会消耗更多内存。

### int(20)中20的涵义

是指显示字符的长度。20表示最大显示宽度为20，但仍占4字节存储，存储范围不变；
不影响内部存储，只是影响带 zerofill 定义的 int 时，前面补多少个 0。

### drop、delete与truncate的区别

![image-20220922083125276](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220922083125276.png)



## 优化

### 如何定位及优化有性能问题的SQL语句(慢查询)

1. 开启慢查询日志。运行时动态设置即可：

   ```sql
   mysql> set global slow_query_log = on;
   mysql> set @@global.long_query_time=2;
   ```

2. 切到/var/lib/mysql/目录查看慢日志文件名，如：7a67d1ef2cb8-slow.log

3. 使用**mysqldumpslow** 工具获取最慢的前10条sql查询 (含左连接的sql)

   `mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/7a67d1ef2cb8-slow.log`

4. 对查询到的sql语句，依次执行 explain 操作，查看详细的执行计划，依次查看以下字段。

   ![image-20220922143435899](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220922143435899.png)

   1.  **id：** 数值大的先执行，相等的从上往下依次执行
   2.  **=type=：** 性能依次衰减：system>const>eq_ref**([多表join时被驱动表]需要用到唯一索引)**>ref**(需要用到普通索引)**>range**(范围查询且需要用到索引)**>index**(对索引进行了全扫描,可加过滤条件)**>all**(全表扫描,没有用到索引)**
   3.  **=possible keys=：** 可能用到的索引，会陈列所有添加过的，没有的话就**<u>需要添加索引进行优化</u>**。
   4.  **=key=：** 具体使用到了那个索引，没有的话说明该sql未命中索引**<u>需要对sql进行优化</u>**。
   5.  **key_len：** 所使用的索引长度，该值越小效果越好。char(n)[n/3n字节(中文)]、varchar(n)[3n+2]、inyint[1]、smallint[2]、int[4]、bigint[8]。date[3]、timestamp[4]、datetime[8]。
   6.  **ref：**过滤条件的类型
   7.  **rows**：执行查询的行数，数值越小则查询次数越少，效率越高。
   8.  **=Extra=**：
       1.  using index：使用了覆盖索引。
       2.  using where：在server层进行了过滤。**未使用到索引。**
       3.  using index condition：查询的列没有被索引完全覆盖。
       4.  using filesort：无法使用索引排序。**order by 未加索引。**
       5.  using temporary：使用到了临时表。说明无法基于索引进行排序、分组、过滤。**需要尝试用索引来优化**。



### *大表查询如何优化

1. 查看表结构是否合理，字段设计【难改】和值的类型【尚可微调】
2. 限定数据的范围，强制携带范围条件的进行查询。
3. explain检查sql语句，尝试sql优化
4. 加缓存，如redis；
5. 通过主从复制使读写分离，来分摊请求；
6. **垂直拆分**。根据模块的耦合度，将大的系统分为多个小系统，适用于部分列常变，部分列不常变的情况；
   + 可减少行数据，减少查询时读取的Block数，减少I/O次数。能简化表的结构，易于维护。
   +  主键会出现冗余，需要管理冗余列，并会引起Join操作，可以通过在应用层进行Join来解决。此外，垂直分区会让事务变得更加复杂；
7. **水平切分**。会保留表结构，分割行数据，避免单表数据量过大。但是比较鸡肋，因为数据还是在同一台机器上，所以通常会伴随分库。

### 超大分页怎么处理

如：sql = `select * from table where age > 20 limit 1000000,10`

+ 主键值连续：使用where对索引先过滤，`select from table where id > 1000000 limit 10`
+ 主键值连续：强制使用覆盖索引 ，select * from table where id in **sql**;



### 优化关联查询

确保ON或者USING子句要能用到索引。确保GROUP BY和ORDER BY只有一个列，这样MySQL才有可能使用索引

### 优化子查询

### 优化分组、排序查询

### 优化LIMIT分页

对起始行的id进行过滤，来减少索引的扫描。

### 优化WHERE子句

1. 建立索引

2. 避免在where中进行null判断，会放弃使用索引而进行全表扫描。

3. 避免使用负向匹配的操作符，如 !=或<、>、NOT IN...

4. 避免使用or来连接条件，会放弃索引执行全表扫描。

5. 避免在 where 中进行表达式或函数操作。

   

### MySQL数据库cpu飙高怎么处理

1. 先用OS命令 top 判断 是不是 mysqld 占用导致的
2. `show processlist` 查看会话连接情况，是否有大消耗的sql在运行
3. 有的话就kill掉这些线程
4. 对涉及到的sql执行explain操作，拿到执行计划，对sql进行优化
5. 再重跑sql

### MySQL的复制原理以及流程

**主从复制：**  Binlog：主数据库的二进制日志 。 Relay log：从服务器的中继日志

1. 由主库的binlog线程会将所有发生数据改变的操作记录到master上的 binlog中
2. 从库的IO线程会在start slave 之后，会将master上的binlog内容，拉到自己的relay log中
3. 从库的sql执行线程会执行relay log，实现主从库的数据一致。



### MySQL记录binlog的方式

1.  STATEMENT（基于SQL语句的复制）：每一条会修改数据的sql语句都会记录到binlog中。优点：减少了binlog日志量，节约IO，提高性能。
2. ROW（基于行 的复制）：仅记录哪条数据被修改成了什么样子。

3. MIXED（混合模式）：一般的复制使用STATEMENT模式，对于STATEMENT 模式无法复制的操作使用ROW模式。

### ❌ MYSQL的各种锁

**乐观锁：**基于数据版本（Version）号记录机制实现。

```sql
-- 更新value字段
update TABLE
set value=2,version=version+1
where id=#{id} and version=#{version};
```

**悲观锁：**在每次操作前都要通过获取锁才能进行对相同数据的操作。使用前，需要关闭mysql的自动提交，手动开启事务。

```sql
set autocommit=0;
start transaction; 
-- 加锁
select status from TABLE where id=1 for update; 
insert into TABLE (id,value) values (2,2);
update TABLE set value=2 where id=1;
commit;
```

+ 共享锁（读锁）：读锁之下，所有的事务只能读不能写，加锁之后其他事务只能通过再加读锁才能进行读操作。读读不互斥，加上共享锁后，对于 update,insert,delete 语句会自动加排它锁。

  ```sql
  set autocommit=0;
  start transaction; 
  SELECT * from TABLE where id = 1 lock in share mode;
  commit;
  ```

+ 排它锁（写锁）：事务对某一行加上了写锁，只能由该事务对目标行进行写操作。其它事务只能读该行数据，不能进行加锁操作，更不能对该行进行写操作。

  ```sql
  -- 直接加 for update
  select status from TABLE where id=1 for update; 
  ```

**行锁：**

+ 共享锁（读锁）：`SELECT * from TABLE where id = "1" lock in share mode;`
+ 排它锁（写锁）：`select status from TABLE where id=1 for update;`

**表锁：**Innodb 的行锁是依赖索引的，没有索引的话会升级为表锁



### ✔️ *Mysql死锁如何避免和解决



