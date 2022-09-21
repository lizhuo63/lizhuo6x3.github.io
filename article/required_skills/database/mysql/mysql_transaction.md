# 四大特性

+ **原子性：**操作要么同时成功，要么同时失败。即在合适的时候回滚，依赖于 **undo.log**
+ **持久性：**事务一旦提交，对数据库的改变就应该是永久性的。防止异常退出之后数据的改动，由 **redo.log** 实现。
+ **隔离性：**事务「并发」执行时，彼此的操作不能互相干扰。否则就会产生脏读、重复读、幻读的问题。为此InnoDB引擎定义了四种隔离级别
  + read uncommit (**读未提交**)：可以看到事务提交之前的数据  --> <u>**脏读**</u>
  + read commit (读已提交)：数据在提交后才能被看到，两次看到的数据不一致  --> **<u>不可重复读</u>**【主要涉及到数据的删改】
  + repeatable read (**可重复复读**)：数据随时能被访问，但在事务之后冒出来原本就没有的新数据。  --> **<u>幻读</u>**【主要涉及数据的添加】
  + serializable (**串行**)：同一行记录， “ 写 ” 会加 “ 写锁 ” ， “ 读 ” 会加 “ 读锁 ” 。读写都串行执行。
+ **一致性：**保障数据变动前后信息的一致性，这是事务的目的。

![image-20220630164848592](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220630164848592.png)

**<u>幻读问题由间隙锁解决</u>**



# 事务类型

**隐式事务，**事务没有明显的开启和结束的标记，比如insert、update、delete语句。

**显式事务，**事务具有明显的开启和结束的标记，需要设置手动提交`set autocommit=0;`

+ 查看当前的隔离级别：`SELECT @@tx_isolation;`
+ 设置当前连接的隔离级别：`SET TRANSACTION ISOLATION LEVEL READ COMMITTED;`
+ 设置全局系统的隔离级别：`SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;`

# 事务的执行

+ **开启事务**

1. 增删改数据会自动开始事务
2. 手动开启事务 begin / start transaction

+ **结束事务**

1. 提交
2. 回滚
3. 连接断开

# InnoDB事务原理 LBCC+MVCC

## MVCC

​			MCVV (多版本并发控制)仅作用于读已提交、可重复读这两种隔离级别，基于数据版本提供不同的可见性，已解决不同事务的读一致性问题。

**要点1. InnoDB会为每行记录实现三个隐藏字段**

![image-20220630180426090](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220630180426090.png)

+ DB_ROW_ID ：行标识 
+ DB_TRX_ID ：自增，记录执行插入或更新行的最后一个事务的ID，即提供当前最新版数据的事务ID
+ DB_ROLL_PTR ：一个版本指针，指向undo.log

DB_ROLL_PTR最终会形成一条undo.log链，保存有事务ID和数据版本号以及具体行数据的绑定关系。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-30_18-22-00.png)

​		为了确保数据版本不丢失，undo.log并不会在更新和回滚之后立即删除，而是确保当前的版本链不再被引用之后才进行删除。

**要点2. ReadView是各事务确定可见性的核心依据**

ReadView是一个数据结构，包含4个字段 

+ m_ids：当前活跃的事务编号集合（未提交、未回滚的）
+ min_trx_id：最小活跃事务编号 
+ max_trx_id ：预分配的事务编号，当前最大事务编号+1 
+ creator_trx_id：ReadView创建者的事务编号 

​	判定标准：从新版本到旧版本，依次取版本链中的DB_TRX_ID

1. DB_TRX_ID是否等于creator_trx_id --> 读取自身事务的数据
2. DB_TRX_ID是否小于min_trx_id --> 读取已提交事务的数据
3. DB_TRX_ID是否大于max_trx_id  --> 禁止读取在当前事务开启之后才开启的事务数据
4. DB_TRX_ID是否介于min_trx_id和max_trx_id 之间，并且不在m_ids的集合内 --> 禁止读取未提交的事务数据

​	**针对于读已提交：**

每次seclet操作都会生成一个ReadView

​	**针对可重复读：**

仅第一次select操作时生成一个ReadView，后续会复用。可解决幻读问题。



## LBCC(锁)

### 行锁

​		通过扫描命中索引来对指定的行加锁，如果未能命中就会进行整表扫描，间接就像是给整张表上了锁。所以说行锁是基于索引加的锁，如果在更新操作时，条件索引失效，那么行锁也会升级为表锁。另外，在批量更新操作时，行锁就很可能会升级为表锁。

#### 共享锁

又称读锁、S锁。即多个事务对同一行数据的访问可以共享一把锁，实现并发读，但不支持并发写。

加锁：`select语句 + LOCK IN SHARE MODE; `

释放锁：事务结束。

#### 排他锁

又称写锁、X锁。该锁无法与其他任何锁共存，只有获取排他锁的事务才允许对行数据进行读和写。

加锁：

自动：delete/update/insert默认加上X锁； 

手动：`sql语句 + FOR UPDATE; `

释放锁：事务结束。

![image-20220701075135314](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220701075135314.png)

**索引记录**：行上有数据的索引

**索引间隙**：临近两个记录的中间开合区域

**索引临键**：左开右闭的索引间隙

#### 记录锁

用于锁定确定目标（索引记录），常用于唯一索引的等值查询。【精准匹配】如上图的id=4,会锁定该索引记录

手动：`sql语句 + FOR UPDATE; `

#### 间隙锁

用于锁定区域 (sql条件所处的 **索引临键** ) ，【范围阻断】如上图id=5 -> 锁住**(4,7]**；id>15 -> 锁住(10,+n)；5<id<8 -> 锁住**(4,10]**

手动：`sql语句 + FOR UPDATE; `

### 表锁

意向共享锁

意向排他锁



















