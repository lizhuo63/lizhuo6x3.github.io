

# 三大范式

+ **第一范式**：要求表的每一列都是不可再分割的原子数据项。
+ **第二范式**：确保表中的每一列都和主键（整体）相关，而不能只与主键的某一部分相关（主要针对联合主键而言）。**要求标识具有唯一性**
+ **第三范式**：确保表中的每一列数据都和主键直接相关，而不是间接相关。**要求字段没有冗余**

# 数据类型

**使用方针：**

1. 确定数据类型
2. 确定具体形式

**尽量选小的，尽量设NOT NULL**

## 数值型

+ 显示长度如INT(11)没有优化效果，在配合使用ZEROFILL时有0填充。
+ ID号尽量选无符号类型，具体根据体量而定
+ DECIMAL适合用于需要计算的数据

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-18_21-10-01.png)

### 时间型

+ TIMESTAMP是UTC时间戳，与时区相关；而DATATIME与时区无关，数据在存取上是一致的。
+ TIMESTAMP比DATETIME更节约空间，无上限要求尽量使用TIMESTAMP。<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-19_10-50-50.png" style="zoom:150%;" />

### 文本型

**CHAR的存储空间都是一次性分配的，不存在碎片的困扰。而使用varchar时，因为存储的长度是可变的，当数据长度在更改前后不一致时，就不可避免地会出现碎片的问题。**

+ 变长比定长类型更节省空间，因为它仅使用必要的空间
+ VARCHAR 需要使用1或2个额外字节记录字符串的长度：如果列的最大长度小于或等于255字节，则只使用1个字节表示，否则使用2个字节。频繁UPDATE会产生空间碎片
+ VARCHAR（50）和VARCHAR（200）存储同一个数据的硬盘空间开销是相同的，但内存开销不同。
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

# SQL语句

## Data Definition Language (DDL)数据定义

支持对库、表、表索引的建删改和清空。常用的语句关键字有 CREATE、DROP、ALTER、TRUNCATE 等

### 库

```sql
CREATE DATABASE 数据库名;
USE 数据库名;
SHOW DATABASES;
ALTER DATABASE 数据库名 xxx;更改库的元数据；
DROP database 数据库名;
```

### 表

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

### 列

```
SHOW COLUMNS FROM 表名;
ALTER TABLE <表名> ADD <新列名> <列定义>; # 添加列
ALTER TABLE <表名> CHANGE <旧列的名字> <新列的名字> <新列的数据类型>; # 可改列名和数据类型
ALTER TABLE 表名 MODIFY 列名 列定义; # 只能改数据类型
ALTER TABLE <表名> DROP <列名>
```

### 约束

+ 主键：`字段名 数据类型 primary key;` ，`primary key(id)`

+ 外键：

  设置外键：`CONSTRAINT 外键名 FOREIGN KEY (从表外键字段) REFERENCES 主表 (主键字段)`，

  添加外键：`ALTER TABLE 从表名 ADD CONSTRAINT 外键名 FOREIGN KEY (从表外键字段) REFERENCES 主表 (主键字段);`

  删除外键：`alter table 从表名 drop foreign key 外键名；`

+ 非空：`字段名 数据类型 NOT NULL;`

+ 默认值：`字段名 数据类型 DEFAULT 默认值；`

+ 唯一性：`字段名 数据类型 UNIQUE;`

## Data Manipulation Language(DML)数据操作

支持对数据进行增删改操作。常用的语句关键字有 INSERT、UPDATE、DELETE 等。

```
# 增
INSERT INTO 表名(字段名1,字段名2...) VALUES(值1，值2...),(值1，值2...)...(值1，值2...);
# 清空表
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

## Data Query Language(DQL)数据查询

### DQL完整语法

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

### 常见单表用法

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

### 多表用法

#### 内连接

`select [columns] from 表1 join 表2 on [condition] where [其它条件];`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-07-35.png" style="zoom:33%;" />

#### 左外连接

**含交：**

`select [columns] from 表1 left join 表2 on [condition] where [其它条件];`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-10-31.png" style="zoom: 50%;" />

**舍交：**

`select [columns] from 表1 left join 表2 on [condition] where 某列 is null;`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-10-54.png" style="zoom: 50%;" />

#### 右外连接

**含交：**

`select [columns] from 表1 right join 表2 on [condition] where [其它条件];`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-14-47.png" style="zoom:50%;" />

**舍交：**

`select [columns] from 表1 right outer join 表2 on [condition] where 某列 is null;`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-15-49.png" style="zoom:50%;" />

#### 全外连接

**含交：**

`select column_list from 表1 left join 表2 on 条件 union select column_list from 表1 right join 表2 on 条件`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-16-38.png" style="zoom:50%;" />

**舍交：**

`select column_list from 表1 left join 表2 on 条件 where 列A is null union select column_list from 表1 right join 表2 on 条件 where 列A is null`

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-28_12-15-28.png" style="zoom:50%;" />

#### 自连接：

```sql
select e1.ename, e2.ename 
from t_employee e1, t_employee e2 
where e1.mid=e2.eid;
```

#### 子查询：

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

## Data Control Language(DCL)数据控制

支持设置/更改数据库用户权限。常用关键字有 GRANT、REVOKE 等。

### 用户

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

### 授权

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

# 事务

## 四大特性

+ **原子性：**操作要么同时成功，要么同时失败。
+ **隔离性：**事务「并发」执行时，彼此的操作不能互相干扰。否则就会产生脏读、重复读、幻读的问题。为此InnoDB引擎定义了四种隔离级别
  + read uncommit(读未提交)：可以看到事务提交之前的数据  --> <u>脏读</u>
  + read commit (读已提交)：数据在提交后才能被看到  --> <u>不可重复读</u>
  + repeatable read (可重复复读)：指事务执行过程中看到的数据总是跟该事务在启动时看到的数据是一致的。  --> <u>幻读</u>
  + serializable (串行)：同一行记录， “ 写 ” 会加 “ 写锁 ” ， “ 读 ” 会加 “ 读锁 ” 。读写都串行执行。
+ **持久性：**事务一旦提交，对数据库的改变就应该是永久性的。数据的持久化，由redo log实现。
+ **一致性：**保障数据变动前后信息的一致性，这是事务的目的。

## **隐式事务**

隐式事务，事务没有明显的开启和结束的标记，比如insert、update、delete语句。

## **显示事务**

显式事务，事务具有明显的开启和结束的标记，需要设置手动提交`set autocommit=0;`

+ 查看当前的隔离级别：`SELECT @@tx_isolation;`
+ 设置当前连接的隔离级别：`SET TRANSACTION ISOLATION LEVEL READ COMMITTED;`
+ 设置全局系统的隔离级别：`SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;`

# 索引的使用

## 索引设计原则

+ 对于经常查询的字段，建议创建索引。

+ 避免对经常更新的表设置过多的索引。因为当表中数据更改的同时，索引也会进行调整和更新，十分消耗系统资源。

+ 数据量小的表建议不要创建索引。数据量小时索引不仅起不到明显的优化效果，对于索引结构的维护反而消耗系统资源。

+ 区分度低的字段不建议建立索引。比如性别字段，只有 “男” 和 “女” ，建索引完全起不到优化效果。

+ 唯一索引能提高查询速度。

+ 在频繁进行排列分组（ group by、order by）的列上建立索引，如果待排序有多个，可以在这些列上建立组合索引。

## 索引失效情况

+ **未明确范围、或者范围很大**：`select name from s1;`，`select name from s1 wehere id > 500;`
+ **like以%开头**

+ **索引列不独立：** sql语句中的索引列必须是独立出现的，`where id + 1 = 5`就不行。
+ **函数不能命中索引：**`select * from tb1 where reverse(email) = 'duoduo'; `
+ **where,group by,order by字段没有设置索引**
+ **数据类型不一致：**`select * from tb1 where name= 999;`
+ **text类型未指定长度：**`create index xxxx on tb(title(19))`



+ **索引列不完整：**主要出现在联合索引中

+ **or的联合条件索引不全：**每个条件列都要有索引

+ **联合索引绑定的某一列没有出现在查询条件中：**（索引顺序）该列及之后的列索引均失效

+ **<u>多个or，按联合索引的顺序从左到右进行查找；多个and，按and顺序从左至右进行查找。</u>**

# SQL优化

## 建表优化

+ 索引优先考虑where、order by使用到的字段。
+ 尽量使用数字型字段（如性别，男：1 女：2），字符型会降低查询和连接的性能，增加存储开销。这是因为引擎在处理查询和连接时会 逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。
+ 用变长型数据代替定长类型。可以省空间、在小字段内检索的效率更高。
+ UNSIGNED表示不允许负值，大致可以使正数的上限提高一倍。
+ schema的列不要太多。如果列太多而实际使用的列又很少的话，有可能会导致CPU占用过高。
+ 大表ALTER TABLE非常耗时，可以创建一个张空表，从旧表中查出所需的数据插入新表，然后再删除旧表。

## SQL查询

### 通用

+ **避免使用select ***，这会让优化器无法完成索引覆盖扫描，会影响优化器对执行计划的选择，增加网络带宽消耗，更会带来额外的I/O,内存和CPU消耗。
+ **调整Where中的连接顺序**。MySQL采用从左往右，自上而下的顺序解析where子句。将过滤数据多的条件往前放，能最快速度缩小结果集。
+ **union查询：**MySQL通过创建并填充临时表的方式来执行union查询。union会给临时表加上distinct，对整个临时表的数据做唯一性校验，此时的消耗相当高。若没有distinct需求则建议使用union all。如：`SELECT ... UNION ALL SELECT ... ;`
+ 将复杂SQL**拆分**为多个小SQL，避免大事务，简单SQL更容易访问缓存、减少表锁的时间、提高多核利用效率.。

### 模糊查询

+ 万级以上数据量尽量避免在字段开头模糊查询，会导致数据库引擎放弃索引进行全表扫描。如`SELECT * FROM t WHERE username LIKE '%li'`;参阅《MySQL模糊查询用法大全（正则、通配符、内置函数等）》

### 范围查询

+ 尽量避免使用in 和not in，会导致引擎走全表扫描。如：`SELECT * FROM t WHERE id IN (2,3)`,可使**用between代替**；在子查询情况中可使用exists替代，如：
+ 尽量避免使用 or，会导致数据库引擎放弃索引进行全表扫描。可使用union代替，如：`SELECT * FROM t WHERE id = 1  UNION SELECT * FROM t WHERE id = 3`
+ **分页【大偏移量查询】：**` SELECT FROM user_innodb WHERE id > 9000000 LIMIT 10;` # 先过滤ID（因为ID使用的是索引），再limit。

### 条件查询

+ 尽量避免进行null值的判断，会导致数据库引擎放弃索引进行全表扫描。解决方法：可以给字段设置不合理的默认值，并对该值进行判断。
+ 尽量避免在where条件中等号的左侧进行表达式、函数操作，会导致数据库引擎放弃索引进行全表扫描，如`SELECT * FROM T WHERE score/10 = 6`
+ **联合索引：**联合索引涉及到的列必须都出现在条件中，否则会全表扫描。
+ **类型转换：**当所给条件与建表的列类型不符时（自行转换），会全表扫描。如：`select col1 from table where col_varchar=123; `
+ 对于复杂的查询，可以使用中间临时表暂存数据；

### 排序

+ order by 的列要与where中列一致，否则不会利用索引进行排序，如：`SELECT * FROM t order where sorce between 60 and 80 by age;`

### 分组

默认情况下MySQL会对GROUP BY分组的所有值进行排序，如 `GROUP BY col1,col2 ;`等同于 `GROUP BY col1,col2 ORDER BY col1,col2;` 因此，如果分组后不想对分组的值进行排序，你可以指定 ORDER BY NULL禁止排序。例如：`GROUP BY col1,col2 ORDER BY NULL`;

### 多表

+ 数据量越小的表越往左排。MySQL执行表关联查询是从左往右执行的（Oracle相反），第一张表会涉及到全表扫描，所以将小表放在前面，扫描快效率更高。
+ 使用表的别名可以减少解析的时间，并减少哪些有列名歧义引起的语法错误。
+ 用where替换HAVING。HAVING在得到检索结果之后进行过滤，而where则是在聚合前过滤记录，通过where限制记录的数目能减少开销。

## DML优化

### 插入

+ 使用`Insert into T values(1,2),(1,3),(1,4);`单语句批量插入，可以减少SQL语句的解析次数、对MySQL server的连接次数、网络IO（语句更短）。

### 删除

+ 使用truncate替代delete会减少占用提高效率，但truncate操作不会被记录到undo块和binlog中，数据不能被恢复。

### 修改

