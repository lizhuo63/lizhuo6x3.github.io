# 官方文档8

[官方文档](https://git-scm.com/book/zh/v2)

# 使用

## 三大范式

+ **第一范式**：要求表的每一列都是不可再分割的原子数据项。
+ **第二范式**：确保表中的每一列都和主键（整体）相关，而不能只与主键的某一部分相关（主要针对联合主键而言）。**要求标识具有唯一性**
+ **第三范式**：确保表中的每一列数据都和主键直接相关，而不是间接相关。**要求字段没有冗余**

## 数据类型

**使用方针：**

1. 确定数据类型
2. 确定具体形式

**尽量选小的，尽量设NOT NULL**

### 数值型

+ 显示长度如INT(11)不会影响数据的插入，只会在使用ZEROFILL时有用，让查询结果前填充0。故正常使用时尽量不适用显示长度。
+ ID号尽量选无符号类型，具体根据体量而定
+ DECIMAL适合用于需要计算的数据

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-18_21-10-01.png)

### 时间型

+ TIMESTAMP是UTC时间戳，与时区相关；而DATATIME与时区无关，数据在存取上是一致的。
+ TIMESTAMP比DATETIME更节约空间，无上限要求尽量使用TIMESTAMP。<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-19_10-50-50.png" style="zoom:150%;" />

### 文本型

**CHAR的于存储空间都是一次性分配的，不存在碎片的困扰。而使用varchar时，因为存储的长度是可变的，当数据长度在更改前后不一致时，就不可避免地会出现碎片的问题。**

+ 变长比定长类型更节省空间，因为它仅使用必要的空间
+ VARCHAR 需要使用1或2个额外字节记录字符串的长度：如果列的最大长度小于或等于255字节，则只使用1个字节表示，否则使用2个字节。频繁UPDATE会产生空间碎片
+ VARCHAR（5） 和VARCHAR（200）存储同一个数据的空间开销是相同的，但更长的列会消耗更多的内存。
+ 使用VARCHAR来UPDATE时，若行占用的空间明显增长，但却没有多的空间可以存储，则MyISAM会将行拆成不同的片段来存储，InnoDB需要分裂页来使行可以放进页内。
+ **使用VARCHAR的场景**：**1.数据长度较长 2.行数据的长度差异不大，不会出现UPDATE后空间不足的情况  3. 不会经常UPDATE
+ **使用CHAR的场景**：1.数据长度较短

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-21_08-30-16.png)

### JSON型

#### 形式

```
# JSON 数组
["abc", 10, null, true, false]
# JSON 对象
{"k1": "value", "k2": 10}
# 嵌套
[99, {"id": "HK500", "cost": 75.99}, ["hot", "cold"]]
{"k1": "value", "k2": [10, 20]}
```

#### 操作

+ 生成

    + 获取json数组：`JSON_ARRAY(1, "abc", NULL, TRUE, CURTIME()) `

    + 获取json对象：`JSON_OBJECT('id', 87, 'name', 'carrot')`

    + 将字符串转化为json：` JSON_QUOTE('[1, 2, 3]')`
    + 归并融合json：`  JSON_MERGE_PRESERVE('{"a": 1, "b": 2}', '{"c": 3, "a": 4}', '{"c": 5, "d": 3}')` =》`{"a": [1, 4], "b": 2, "c": [3, 5], "d": 3}`
    + 覆盖融合json：`JSON_MERGE_PATCH('{"a": 3, "b": 2}', '{"c": 3, "a": 4}', '{"c": 5, "d": 3}')` =》 `{"a": 4, "b": 2, "c": 5, "d": 3}`

+ 搜索

    + 取值：`JSON_EXTRACT('{"id": 14, "name": "Aztalan"}', '$.name') `

        `$.[*]`匹配JSON对象所有成员；$."a"

        `$[*]`匹配JSON数组所有元素；$[1].a[1]，$[1 to 3]，$[last-3 to last]

## SQL语句

### Data Definition Language (DDL)数据定义

支持对库、表、表索引的建删改和清空。常用的语句关键字有 CREATE、DROP、ALTER、TRUNCATE 等

#### 库

```sql
CREATE DATABASE 数据库名;
USE 数据库名;
SHOW DATABASES;
ALTER DATABASE 数据库名 xxx;更改库的元数据；
DROP database 数据库名;
```

#### 表

```
CREATE TABLE 表名(
列名 数据类型 约束,
列名 数据类型 约束
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

SHOW TABLES;
SHOW CREATE TABLE 表名;
ALTER TABLE 表名 RENAME TO 新表名; # 改表名
CREATE TABLE  <新表名>  LIKE  <旧表>; # 复制表结构
CREATE TABLE  <新表名>  AS  (SELECT  *  FROM  <旧表名>); # 复制表结构+数据
DORP TABLE 表名;
```

#### 列

```
SHOW COLUMNS FROM 表名;
ALTER TABLE <表名> ADD <新列名> <列定义>; # 添加列
ALTER TABLE <表名> CHANGE <旧列的名字> <新列的名字> <新列的数据类型>; # 可改列名和数据类型
ALTER TABLE 表名 MODIFY 列名 列定义; # 只能改数据类型
ALTER TABLE <表名> DROP <列名>
```

#### 约束

+ 主键：`字段名 数据类型 primary key;` ，`primary key(id)`

+ 外键：

    设置外键：`CONSTRAINT 外键名 FOREIGN KEY (从表外键字段) REFERENCES 主表 (主键字段)`，

    添加外键：`ALTER TABLE 从表名 ADD CONSTRAINT 外键名 FOREIGN KEY (从表外键字段) REFERENCES 主表 (主键字段);`

    删除外键：`alter table 从表名 drop foreign key 外键名；`

+ 非空：`字段名 数据类型 NOT NULL;`

+ 默认值：`字段名 数据类型 DEFAULT 默认值；`

+ 唯一性：`字段名 数据类型 UNIQUE;`



### Data Manipulation Language(DML)数据操作

支持对数据进行增删改操作。常用的语句关键字有 INSERT、UPDATE、DELETE 等。

```
# 增
INSERT INTO 表名(字段名1,字段名2...) VALUES(值1，值2...),(值1，值2...)...(值1，值2...);
# 删
TRUNCATE TABLE 表名;
DELETE FROM 表名 WHERE 筛选条件;
# 改
UPDATE 表名 SET 字段名1=值1,字段名2=值2... WHERE 筛选条件;

DELETE和TRUNCATE的区别：
① DELETE 可以添加WHERE条件,TRUNCATE不能，直接清空
② TRUNCATE 的效率较高，不需要逐行判断
③ 如果删除带自增长列的表，使用DELETE后，记录从断点处开始；使用TRUNCATE后则从1开始。
④ DELETE删除数据返回受影响的行数,TRUNCATE不返回
⑤ DELETE支持事务回滚,TRUNCATE不支持
```

**复制表结构：**`create table 表名 like 被复制的表名;`

**复制表结构和数据：**`create table 表名 as (select * from 被复制的表名);`

### Data Query Language(DQL)数据查询

支持对数据进行查询操作。常用关键字有 SELECT、FROM、WHERE 等。

#### DQL完整语法

```sql
SELECT DISTINCT
	<select_list>
FROM
	<left_table> <join_type>
JOIN <right_table> ON <join_condition>
WHERE
	<where_condition>
GROUP BY
	<group_by_list>
HAVING
	<having_condition>
ORDER BY
	<order_by_condition>
LIMIT <limit_number>

```

#### 常见单表用法

```sql
# ===单表===
select pname as pn from product where id =1;
# 去重
select distinct price from product;
# 条件
	select * from student where age>15 or gender='male';
	select * from student where sid in ('S_1002','S_1003'); # 集合
	select * from student where age between 15 and 18; # 范围
	select * from student where sname is not null; # 空值
# 计算表达式
select pname, price+10 from product;
# 函数 
select avg(price) from product;
# 模糊
select * from student where sname like 'li%';
# 分组
select cid, avg(price) from product group by cid having avg(price) > 60;
# 排序
select * from student order by age asc;
# 分页
select eid,ename,gender from t_employee limit 4, 2; #【（页码-1）*行数，行数】

```

#### 多表用法

##### 内连接

`select [columns] from 表1 join 表2 on [condition] where [其它条件];`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-07-35.png" style="zoom:33%;" />

##### 左外连接

**含交：**

`select [columns] from 表1 left join 表2 on [condition] where [其它条件];`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-10-31.png" style="zoom: 50%;" />

**舍交：**

`select [columns] from 表1 left join 表2 on [condition] where 某列 is null;`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-10-54.png" style="zoom: 50%;" />

##### 右外连接

**含交：**

`select [columns] from 表1 right join 表2 on [condition] where [其它条件];`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-14-47.png" style="zoom:50%;" />

**舍交：**

`select [columns] from 表1 right outer join 表2 on [condition] where 某列 is null;`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-15-49.png" style="zoom:50%;" />

##### 全外连接

**含交：**

`select column_list from 表1 left join 表2 on 条件 union select column_list from 表1 right join 表2 on 条件`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-16-38.png" style="zoom:50%;" />

**舍交：**

`select column_list from 表1 left join 表2 on 条件 where 列A is null union select column_list from 表1 right join 表2 on 条件 where 列A is null`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-15-28.png" style="zoom:50%;" />

##### 自连接：

```sql
select e1.ename, e2.ename 
from t_employee e1, t_employee e2 
where e1.mid=e2.eid;
```

##### 子查询：

```sql
select eid,ename,dept_id
from t_employee
where dept_id in (
    select dept_id
    from t_employee
    where ename in ('userA','userB')
);
```

+ where型子查询：指把内部查询的结果作为外层查询的比较条件。

  ```sql
  select name from goods where id in(select id from goods ...);
  ```

+ from型子查询：把内层的查询结果当成临时表，供外层sql再次查询。

+ in子查询：内层查询语句仅返回一个数据列，这个数据列的值将供外层查询语句进行比较

+ exists子查询：把外层的查询结果，拿到内层，看内层是否成立，简单来说后面的返回true,外层（也就是前面的语句）才会执行，否则不执行。

+ any子查询：只要满足内层子查询中的任意一个比较条件，就返回一个结果作为外层查询条件。

+ all子查询：内层子查询返回的结果需同时满足所有内层查询条件。

+ 比较运算符子查询：子查询中可以使用的比较运算符如 “>” “<” “= ” “!=”
  

### Data Control Language(DCL)数据控制

支持设置/更改数据库用户权限。常用关键字有 GRANT、REVOKE 等。

#### 用户

**创建用户：**

```sql
CREATE USER 'alian'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
flush privileges;
```

**修改用户名：**

```sql
rename user '用户名'@'IP地址' to '新用户名'@'IP地址';
```

**修改密码：**

```sql
ALTER USER '用户名'@'IP地址' IDENTIFIED WITH mysql_native_password BY '新密码';
flush privileges;

#  或者普通用户登录后
SET PASSWORD=password('新密码');
flush privileges;
```

**删除用户：**

```sql
drop user '用户名'@'IP地址';
```

#### 授权

**基本语法：**

```sql
grant 权限1, 权限2, 权限3,… ,权限n on 数据库名.表名 to 用户名@地址;
```

**说明：**

+ **mysql.*** 表示mysql库的任意表
+ **’alian’@'localhost’** ：表示只允许本机登录
+ **’alian’@’%'** ：表示任意地址登录
+ **’alian’@'192.168.\*.\*'** ：表示只允许ip为192.168网段的地址登录

```sql
#把mysql数据库的user表的（查询，插入，更新，删除）的权限都给alian,并且是任意ip地址都可以操作
grant SELECT, INSERT, UPDATE, DELETE on mysql.user to 'alian'@'%';
flush privileges;
```

**查看权限：**

```sql
show grants for 'alian'@'%';
```

**回收权限：**

```sql
revoke 权限1, 权限2…权限n on 数据库名.表名 from 用户名@地址;
```

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

# 事务

## 四大特性

+ **原子性：**操作要么同时成功，要么同时失败。
+ **隔离性：**事务「并发」执行时，彼此的操作不能互相干扰。否则就会产生脏读、重复读、幻读的问题。为此InnoDB引擎定义了四种隔离级别
  + read uncommit(读未提交)：可以看到事务提交之前的数据
  + read commit (读已提交)：数据在提交后才能被看到
  + repeatable read (可重复复读)：指事务执行过程中看到的数据总是跟该事务在启动时看到的数据是一致的。
  + serializable (串行)：同一行记录， “ 写 ” 会加 “ 写锁 ” ， “ 读 ” 会加 “ 读锁 ” 。读写都串行执行。
+ **持久性：**事务一旦提交，对数据库的改变就应该是永久性的。数据的持久化，由redo log实现。
+ **一致性：**保障数据变动前后信息的一致性，这是事务的目的。











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

### 具体手段

#### 通用

+ **避免使用select ***，这会让优化器无法完成索引覆盖扫描，会影响优化器对执行计划的选择，增加网络带宽消耗，更会带来额外的I/O,内存和CPU消耗。
+ **调整Where中的连接顺序**。MySQL采用从左往右，自上而下的顺序解析where子句。将过滤数据多的条件往前放，能最快速度缩小结果集。
+ **union查询：**MySQL通过创建并填充临时表的方式来执行union查询。union会给临时表加上distinct，对整个临时表的数据做唯一性校验，此时的消耗相当高。若没有distinct需求则建议使用union all。如：`SELECT ... UNION ALL SELECT ... ;`
+ 将复杂SQL**拆分**为多个小SQL，避免大事务，简单SQL更容易访问缓存、减少表锁的时间、提高多核利用效率.。

#### 模糊查询

+ 万级以上数据量尽量避免在字段开头模糊查询，会导致数据库引擎放弃索引进行全表扫描。如`SELECT * FROM t WHERE username LIKE '%li'`;参阅《MySQL模糊查询用法大全（正则、通配符、内置函数等）》

#### 范围查询

+ 尽量避免使用in 和not in，会导致引擎走全表扫描。如：`SELECT * FROM t WHERE id IN (2,3)`,可使**用between代替**；在子查询情况中可使用exists替代，如：
+ 尽量避免使用 or，会导致数据库引擎放弃索引进行全表扫描。可使用union代替，如：`SELECT * FROM t WHERE id = 1  UNION SELECT * FROM t WHERE id = 3`
+ **分页【大偏移量查询】：**` SELECT FROM user_innodb WHERE id > 9000000 LIMIT 10;` # 先过滤ID（因为ID使用的是索引），再limit。

#### 条件查询

+ 尽量避免进行null值的判断，会导致数据库引擎放弃索引进行全表扫描。解决方法：可以给字段设置不合理的默认值，并对该值进行判断。
+ 尽量避免在where条件中等号的左侧进行表达式、函数操作，会导致数据库引擎放弃索引进行全表扫描，如`SELECT * FROM T WHERE score/10 = 6`
+ **联合索引：**联合索引涉及到的列必须都出现在条件中，否则会全表扫描。
+ **类型转换：**当所给条件与建表的列类型不符时（自行转换），会全表扫描。如：`select col1 from table where col_varchar=123; `
+ 对于复杂的查询，可以使用中间临时表暂存数据；

#### 排序

+ order by 的列要与where中列一致，否则不会利用索引进行排序，如：`SELECT * FROM t order where sorce between 60 and 80 by age;`

#### 分组

默认情况下MySQL会对GROUP BY分组的所有值进行排序，如 `GROUP BY col1,col2 ;`等同于 `GROUP BY col1,col2 ORDER BY col1,col2;` 因此，如果分组后不想对分组的值进行排序，你可以指定 ORDER BY NULL禁止排序。例如：`GROUP BY col1,col2 ORDER BY NULL`;

#### 多表

+ 数据量越小的表越往左排。MySQL执行表关联查询是从左往右执行的（Oracle相反），第一张表会涉及到全表扫描，所以将小表放在前面，扫描快效率更高。
+ 使用表的别名可以减少解析的时间，并减少哪些有列名歧义引起的语法错误。
+ 用where替换HAVING。HAVING在得到检索结果之后进行过滤，而where则是在聚合前过滤记录，通过where限制记录的数目能减少开销。

## DML优化

### 插入

+ 使用`Insert into T values(1,2),(1,3),(1,4);`单语句批量插入，可以减少SQL语句的解析次数、对MySQL server的连接次数、网络IO（语句更短）。

### 删除

+ 使用truncate替代delete会减少占用提高效率，但truncate操作不会被记录到undo块和binlog中，数据不能被恢复。

### 修改



## 建表优化

+ 索引优先考虑where、order by使用到的字段。
+ 尽量使用数字型字段（如性别，男：1 女：2），字符型会降低查询和连接的性能，增加存储开销。这是因为引擎在处理查询和连接时会 逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。
+ 用变长型数据代替定长类型。可以省空间、在小字段内检索的效率更高。
+ UNSIGNED表示不允许负值，大致可以使正数的上限提高一倍。
+ schema的列不要太多。如果列太多而实际使用的列又很少的话，有可能会导致CPU占用过高。
+ 大表ALTER TABLE非常耗时，可以创建一个张空表，从旧表中查出所需的数据插入新表，然后再删除旧表。
