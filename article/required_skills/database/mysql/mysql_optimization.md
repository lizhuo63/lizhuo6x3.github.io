# 执行流程回顾

![image-20220701092433062](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220701092433062.png)

![image-20220702100815998](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220702100815998.png)

# 连接阶段(架构层次)

## 服务端

+ **增加连接数：** 

  + 查看：`SHOW VARIABLES LIKE '%MAX_CONNECTIONS%';`（默认151）

  + 临时会话设置：`set GLOBAL max_connections = 200;`

  + 全局设置：

    ```
    [mysqld]
    max_connections=200
    ```

    `service mysql restart`

+ **缩短连接的超时回收时间：**

  + 查看：`SHOW VARIABLES LIKE '%wait_timeout%';`（默认28800 [8小时]）

  + 临时设置：`set GLOBAL wait_timeout= 3600;`

  + 全局设置：

    ```
    [mysqld]
    wait_timeout=3600
    ```

    `service mysql restart`

+ **做集群，主从复制->读写分离**

+ **分库分表**

## 客户端

+ **使用连接池：**如 druid
+ **连接削峰：**MQ
+ **引入第三方缓存：**redis



# 优化执行阶段(SQL层次)

## SQL和索引的优化(慢查询日志)

开启慢查询日志是有性能代价的，因此MySQL默认是关闭的。慢查询日志是MySQL提供的sql优化的工具。认定执行时间超过`long_query_time`设定的阈值（默认10s）就是慢的，需要被优化的。慢查询被记录在慢查询日志里，慢查询日志默认是不开启的。如果需要优化SQL语句，就可以开启这个功能。

1. 查看当前慢查询状态： `show variables like 'slow_query%';`

2. 更改慢查询阈值：`show variables like '%long_query%';`，默认10s

   ```sql
   # 开启慢查询日志
   [mysqld]
   slow_query_log=ON
   long_query_time=2 # 阈值
   slow_query_log_file=/var/lib/mysql/slow.log
   # 重启
   
   # 动态修改
   mysql> set global slow_query_log = on;
   mysql> set @@global.long_query_time=2;
   ```



### 操作方法：

1. 直接查看慢查询日志：`less /var/lib/mysql/localhost-slow.log`，效率低，不推荐

   ![image-20220702082727864](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220702082727864.png)

2. 直接使用慢查询分析工具：**mysqldumpslow  --help**

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

   ![image-20220702083411171](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220702083411171.png)

   ![image-20220702083741984](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220702083741984.png)

3. 对症下药

   1. 环境问题：

      + 所有参数：`Show global status;`

      + 查看线程(对应客户端的连接)状况：`show processlist`，应当干掉大部分sleep的连接

        运行`show full processlist`查看MySQL中运行的所有线程，将有问题的直接通过id来kill。

        **Id**：线程的唯一标志

        **User**：启动该线程的用户，普通用户只能查看自己的线程

        **Host**：哪个ip和端口发起的连接

        **db**：线程操作的数据库

        **Command**：线程的命令

        **Time**：操作持续时间

        **State**：线程的状态

        **Info**：SQL语句的前100个字符

      + 查看存储引擎状态：`show engine innodb status;`

   2. 优化SQL语句：分析sql语句：`explain + 目标SQL语句` / `explain format=json + 目标SQL语句` / `optimizer trace;`打开路径选择显示

      **explain输出格式：**[内容详解](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)

      ![image-20220702090140290](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220702090140290.png)

      + **id：**从大到小进行查询；若id一样，则从上往下查。（先查外层表，在查内层表）
      + **type：** 性能依次衰减：system>const>eq_ref**([多表join时被驱动表]需要用到唯一索引)**>ref**(需要用到普通索引)**>range**(范围查询且需要用到索引)**>index**(对索引进行了全扫描,可加过滤条件)**>all**(全表扫描,没有用到索引)**
      + **possible keys：**可能用到的索引，没有的话应该加上一个索引
      + **key：**具体使用的索引
      + **ref：**过滤条件的类型
      + **rows：**预估返回行数，InnoDB不精确，但能起到参考作用。
      + **filtered：** 存储引擎对返回结果的过滤率，因为存储引擎能用到索引，所以希望该值尽量大
      + **extra：**
        + using index：使用了覆盖索引。
        + using where：在server层进行了过滤。
        + using index condition：使用量索引下推。查询条件中的索引用不上且优化器判定在引擎层过滤效果更好，就会让引擎层先过滤，再返回结果。
        + using filesort：无法使用索引排序。**order by 未加索引。**
        + using temporary：使用到了临时表。说明无法基于索引进行排序、分组、过滤。**建立索引。**

   3. 可将未使用索引的SQL录入慢查询日志

      ```sql
      show variables like 'log_queries_not_using_indexes'; # 查看设置，默认关闭
      set global log_queries_not_using_indexes = on; # 设置
      ```

   4. [可借鉴官方的优化建议](https://dev.mysql.com/doc/refman/5.7/en/select-optimization.html)。当目标SQL过于复杂时，可进行条件增删、顺序调整、子查询抽离的方式定位问题原因。

   



## 表结构和存储引擎优化









# 附录：

## SQL语句执行顺序

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