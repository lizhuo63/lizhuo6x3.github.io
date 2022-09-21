# 宏观蓝图

![image-20220716162548147](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220716162548147.png)

图虽然是这么画的，但实际上略有出入。该图体现了各个架构的分工，但是在构建集群时，往往会根据搭建的指令自动的构建出主从架构。

# 主从复制

诞生背景，为了实现高可用，防止因一台服务器挂掉而影响整个服务，将多个服务器捆绑起来，即使挂了一个后，其他的服务器也能用，保证服务的可靠性。

**实现：**

+ 服务器互联
+ 数据同步

**master：**主节点，提供写服务；**slave：**从节点，提供读服务；执行写操作时，会将出现变化的数据自动同步到slave。

当其中一个从机宕机了，服务正常继续；此外，为了防止主节点关掉导致服务瘫痪，可对主节点做集群。

## 搭建方式

### 命令方式

**两步走：**

1. 启动两个Redis服务器A、B。（计划A主B从）
2. 启动两个客户端a,b
3. 用从b连接从机B，并发送指令`slaveof A.ip A.port`，指明用当服务器连接A服务器，并充当A的从机。即可搭建成功。
4. 检测。用a去存值，后用b去取。成功。

**一步走：**

3. 当用b连接B时加上后缀指令`--slaveof A.ip A.port `，即可在连接的时候同时设置主从。
4. 检测。用a去存值，后用b去取。成功。

**断连：**`slaveof no one`，从机。

### 配置文件方式

在从机的配置里加上：`slaveof IP port`

### 授权访问

master客户端发送命令设置密码；requirepass password

master配置文件设置密码；config set requirepass password；config get requirepass

slave客户端发送命令设置密码；auth password

slave配置文件设置密码；masterauth password

slave启动服务器设置密码；redis-server –a password

## 细节体现

### 连接阶段

1. 从机发送主从连接请求
2. 主节点收到后，回复响应
3. 从节点收到响应后，保存主节点的ip/port，建立主从连接
4. 定时、周期发送ping/pong指令，确认连接可用
5. 如果主节点设置有密码，需进行身份验证
6. 从节点将port发送给主机的，以备数据同步使用。

![image-20220715140914088](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220715140914088.png)

![image-20220715140930976](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220715140930976.png)

### 数据同步阶段

**概念：**

+ 服务器运行ID：每一台服务器每启动一次都会携带不一样的ID，用与验证身份的一致性，类似Cookie。供数据同步使用。

+ 复制缓冲区：一个先进先出的队列，内部存放的是指令字符。供数据的实时同步使用。此外主从端点会分别记录自身指令读取的偏移量，防止连接故障导致指令读取的进度丢失。

+ 复制偏移量：主节点有多个，从机只有一个。用于防止连接故障导致指令读取的进度丢失，可供数据恢复使用。

  ![image-20220716085215535](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220716085215535.png)

**阶段流程：**

在slave初次连接master后，首先会同步master中的所有数据，将自身的数据库状态更新到与主节点一致。这个过程恢复的是点数据，对恢复效率有要求，故采用的是RDB方式。对于在该阶段新增的后续数据就要着重关注实时同步，故后续采用的是AOF方式。

1. 首次数据同步，由于 runid和offset 没有记录值。从机会发送 `psync2 ? -1` 指令，请求全量复制
2. 主节点执行 bgsave ,创建 RDB 数据文件。并通过socket发送 `fullresync runid offset` 全量复制指令和rdb文件 给从机。【当第一个从机连进来时会创建一个缓冲区，用于存放所有从机的数据同步指令。以便在同步时，主机仍能提供服务】
3. 从机清空当前库，并记录 runid和offset 执行RDB数据恢复，【**至此，点数据RDB已经同步成功，完成了全量复制。**后续需要同步1-3之间以及后续期间，用户新set的数据[存在缓冲区的指令]，部分复制。】
4. 从机发送 `psync2 runid offset` 指令，请求新数据的部分复制。
5. 主节点会率先检查 runid与 offset，如果检测不通过就进行1-3的全量复制，通过了就发送 `continue offset` ,执行部分复制。
6. 从节点通过 offset 去复制缓冲区读取指令，并执行AOF的重写优化，恢复数据。后续小数据的恢复都通过AOF部分复制实现，若出现网络故障，导致 offset 不在索引范围内【期间出现了大量指令，把它挤出去了】，引起5阶段检测不过，走全量复制。

**【注意】：**

+ 当主节点的数据量很大时，要避免在高峰期进行同步
+ 缓冲区是一个队列，如果出现指令塞满了的情况，前面的指令就会丢失，主从数据就无法同步，就会循环陷入全量复制。可通过设置`repl-backlog-size`来调整，适合内存的3到5成。
+ 为避免从机在数据同步时响应阻塞和数据不同步，应该关闭该阶段的对外服务。设置`slave-server-stale-data`

![image-20220715141042535](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220715141042535.png)

![image-20220716083457181](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220716083457181.png)

### 命令传播阶段

**心跳机制**

进入命令传播阶段后，master与slave间需要进行数据同步，使用心跳机制进行通信，保持双方连接在线。当很多的slave离线或延迟过高时【请求全量复制】，master为保障数据稳定性，将拒绝所有数据同步操作；slave数量少于2个或所有slave延迟都大于等于10s时，强制关闭master写功能，停止数据同步。

master心跳： 只是ping一下slave，判断slave是否在线。

slave心跳： slave 每秒发送一次REPLCONF ACK {offset}，汇报自己的复制进度offset，获取最新的数据变更指令，同时判断master是否在线。

**阶段流程：**

1. 从节点发送 `REPLCONF ACK offset`，开始心跳机制

2. 进入数据同步阶段的部分复制环节。

   

## 问题与解决方案

+ 当master重启时，会丢失 runid和offset ,引发所有从机的全量复制
  + 解决：master 在关闭时会执行`shutdown save` 指令，保存runid和offset到RDB文件，重启后加载RDB即可恢复 runid 和offset。
+ 网络不佳，导致offset被挤出缓存队列，引发全量复制
  + 解决：适当加大缓冲区 `repl-backlog-size`
+ 当从机接收到慢查询指令时，将无法及时响应主节点的心跳，引起主节点的频繁确认和从机的断连。造成主节点CPU以及带宽占用过高。
  + 解决：设置合理的超时时间(释放slave的阈值时间) `repl-timeout`
+ 设置的心跳频度与超时时间不合理，导致频繁超时断联
  + 解决：提高ping指令的频度，此外超时时间应设为改值的 5-10 倍。心跳频度：`repl-ping-slave-period`。超时时间`repl-timeout`.



# 哨兵机制

如若主节点挂掉了，那么整个服务就会瘫痪，这是无法容忍的。我们需要从从机中选出一个充当主节点，实现高可用继续淦。

**哨兵（sentinel)** ：本身也是一个Redis服务，但它不提供读写服务，它们合力打造是一个对主从结构中的每台服务器进行监控的**分布式系统**，当出现故障时通过**投票机制选择新的master**，并将所有slave连接到新的master。

## 使用哨兵

**配置文件：**

```
# 哨兵端口
port 26401 
# 哨兵文件存放目录
dir /redis/data 
# 监听的主节点信息
sentinel monitor 自定义的主节点名称 m.ip m.port 认定投票通过的哨兵数量 
# 主节点10秒内未响应从机，就单方面认定改主节点挂了
sentinel down-after-milliseconds 自定义的主节点名称 10000 
# 新的master选出来后，一次和几个slave进行数据同步，该值越小，同步器压力越小，速度越慢
sentinel parallel-syncs 自定义的主节点名称 1
# 设定同步超时时间
sentinel failover-timeout mymaster 10000
sentinel deny-scripts-reconfig yes
```

**启动哨兵：**`redis-sentinel 哨兵配置文件`



## 工作流程

**监控阶段：**

1. 哨兵服务启动后，会率先于主节点之间建立cmd连接，用于高效发送命令。

2. 在哨兵和主节点双方存储共享信息

3. 根据2中的信息去进一步获取主节点的所有从机的信息，至此一个哨兵的所需的信息就获取完毕了。

4. 当有新的哨兵进来，会执行 流程 1

5. 新哨兵得知已有哨兵存在，会同时连接已存在的所有哨兵，用于共享信息。

   ![ ](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220716102421782.png)

**通知阶段：**

1. 哨兵会给所有的对外服务器发送指令，确认状态
2. 将接收到的状态信息在哨兵的内部进行共享，实现信息对等

![image-20220716102703578](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220716102703578.png)

**故障转移阶段：**

1. 当一个哨兵无法接收的主节点的响应时，会认定它挂了，然会将信息共享到其他哨兵
2. 所有哨兵会对该主节点进行确认，当达到指定数量的哨兵判定该主节点挂了，该主节点才最终被标记挂掉了。
3. 哨兵内部会推选一名主事人，进行主节点清退和从机的推选。选举原则：要求在线，要求响应够快，要求与之前的主节点断连时间不能过长。最后根据优先原则选择offset更大的[ 数据相对完整 ]、uuid更小的[ 没办法的办法 ]。
4. 给新的master发送`salve of no one`，断开与原master的主从连接
5. 向其他slave发送新master的ip:port，让其他slave重新确定master



# 集群

目前的方案已经是比较完美了，读写复制可以实现分摊读写请求并且实现数据的冗余备份，但会出现master挂掉导致整个服务瘫痪的严重问题；自此，哨兵机制应运而生，它通过监控所有对外服务器的运行状态，及时将故障的master移除并推选出新的master，构建新的可用的主从架构。

然而目前的问题是，无法实现内存扩容，内存的有效容量完全取决于单机的内存条，无法满足更大业务的需求。所以，出现了将多个集哨兵的主从结构并起来的想法，对外提供一个ip、port就可访问多个主从结构的模型，这就是集群呐。

## 集群的数据存储设计

0. 集群会将所有的内存空间分割成16384份，得到16384个槽并对其标号【数量是定死的，容量是可变的】。当有新节点加入时，当前集群内的所有节点都会匀部分槽位给新节点；当有节点下线时，集群就会回收该节点的槽位分配给剩余的节点。并且每个节点都会保存一份所有节点的槽位标号映射表，供 `get key` 时快速确定目标 key所在的槽位。
1. 用户`set key value`
2. 集群对key进行CRC16(key)，然后 `%16384` 得到一个最终的标记值如 (1056)，以确定该key预计存储的目标槽位
3. 通过槽位映射表，找到目标节点，将key存放到 (1056) 号槽位中
4. 用户 `get key` ，访问节点A时，通过2 的策略找到目标 key 的槽位标号，然后对照槽位映射表，若该槽位在A节点中就直接返回；否则就指引请求到目标节点去找。可实现最多两次查找。

## 实际搭建

1. **配置文件：**

   ```bash
   # 基本配置
   bind 0.0.0.0
   port 6401
   #daemonize yes
   protected-mode no
   #logfile "/opt/redis6/logs/redis-6401.log"
   dbfilename "dump-6401.rdb"
   dir /opt/redis6/data
   
   # aof配置
   save 15 2
   appendonly yes
   appendfilename "appendonly-6401.aof"
   appendfsync everysec
   
   # 集群配置(构建指令会分配slave，故主从无需配置)
   # 设定位集群节点
   cluster-enabled yes
   # 为自动生成的配置文件命名
   cluster-config-file "cluster-6401.conf"
   # 设置响应超时时间，用于判定节点下线
   cluster-node-timeout 15000
   ```

2. **启动服务：**

   ```bash
   redis-server conf/cluster/redis-6401.conf
   ...
   ...-6406.conf
   ```

   

3. **构建集群**

   ```
   # --cluster-replicas 1 ，代表1主配一从
   redis-cli --cluster create --cluster-replicas 1 192.168.154.131:6401 192.168.154.131:6402 192.168.154.131:6403 192.168.154.131:6404 192.168.154.131:6405 192.168.154.131:6406
   ```

4. **连接使用：**`server-cli -c -p 6401` 【'-c' 可实现槽位引导下的节点自动跳转】

   + 查看集群节点信息；cluster nodes

   + 更改slave指向新的master；cluster replicate master-id

   + 发现一个新节点，新增master；cluster meet ip:port

   + 忽略一个没有solt的节点；cluster forget server_id

   + 手动故障转移；cluster failover

   **添加slave**

   **redis-cli --cluster add-node **new-slave-host:new-slave-port master-host:master-port **--cluster-slave  --cluster-master-id** masterid

   **删除slave**

   **redis-cli --cluster del-node **del-slave-host:del-slave-port del-slave-id

   **添加master**

   **redis-cli --cluster add-node** new-master-host:new-master-port now-host:now-port

   **删除master**【保证前提：没有槽位】

   **redis-cli --cluster del-node **del-slave-host:del-slave-port del-slave-id

   **重新分槽**

   **redis-cli --cluster reshard** new-master-host:new-master:port **--cluster-from** src-  master-id1, src-master-id2, src-master-idn **--cluster-to **target-master-id **--cluster-slots** slots

   **归还槽位**

   **redis-cli --cluster reshard **src-master-host:src-master-port **--cluster-from** src-  master-id **--cluster-to** target-master-id **--cluster-slots **slots **--cluster-yes**

   