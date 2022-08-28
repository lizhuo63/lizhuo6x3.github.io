# 缓存梗概

+ 缓存命中时，将不会再取编译，也不会去设参，更不会去执行sql，而是引用已有的返回结果。
+ Mybatrs 有两级缓存，一级缓存有 BaseExecutor 实现，二级缓存由 CacheExecutor 实现。



# 一级缓存

属于会话级别的缓存，范围小，容易被擦除。没有线程干扰**，拿到的是同一个对象，**禁止对其更改，如有必要需对其深拷贝。

## 命中条件

+ sql 和参数必须相同。一旦不同就会查到不同的数据，当然就不应该缓存。
+ 必须有相同的statementID。一级缓存是一个Map结构，它的键就是全类名引用，即statmentID。
+ 没有在查询之间执行更新操作。
+ 必须在一个会话中。
+ RowBounds 的分页范围必须相同。
+ 没有进行手动清空缓存。clearCache( )
+ 没有调用有配置`flushCache=true` 的mapperMethod。
+ 没有手动缩小一级缓存作用域。`<setting name="localCacheScope" value="STATEMENT"/>`

## spring集成Mybatis的一级缓存问题

![image-20220718205645970](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220718205645970.png)

就像这幅图一样，Spring将根据Mapper获取代理Mapper，然后又会获取一个代理的 SqlSessionIntercepter，并根据代理的 SqlSessionIntercepter 去与Mybatis接触，这将导致每调用此次 MapperMethod 都会开启一个新的SqlSession ，直接破坏了Mybatis的一级缓存命中的条件。解决办法就是手动开启事务将多个 MapperMethod 调用包裹起来。



# 二级缓存

属于应用级别的缓存，支持多会话，即能够做到跨线程命中，命中率更高。但是要求对entiry进行序列化。也就是说，二级缓存会拿**到相同的对象副本**【经过序列化】，对于多线程环境来讲，数据上更加安全。**<u>所有对二级缓存的操作必须提交才会生效。</u>**

大生命周期和高命中率带来的问题就是，容易造成大数据缓存导致的OOM，所以要有相对应的数据溢出淘汰策略，即LRU【最近最少使用】。

## Mybatis二级缓存的设计

![image-20220719165220711](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220719165220711.png)

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220719224209567.png)

![image-20220719230051895](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220719230051895.png)

## 二级缓存的命中条件

+ sql 和参数必须相同。一旦不同就会查到不同的数据，当然就不应该缓存。
+ 必须有相同的statementID。
+ **会话必须手动提交或者关闭。**目的是确保当前会话的工作完毕，不会因为事物的回滚导致其他会话出现脏读现象。
+ RowBounds 的分页范围必须相同。
+ 没有进行手动清空缓存。clearCache( )
+ 没有调用有配置`flushCache=true` 的mapperMethod。

## 二级缓存的相关配置

二级缓存默认不开启，如果要使用，需要为其声明缓存空间，可通过`@CacheNamespace`或者<cache/>标签进行配置，以下是他们的**详细配置**。

![image-20220719215558428](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220719215558428.png)

**【 注意：】** 对于二级缓存Mybatis虽然提供了两套配置方案，**XML标签** 和 **注解**，但是对于同一个 XxxMapper 这两个方案无法混用，否则报错。具体看改Mapper接口采用了那种书写Sql的方式 xml对xml ，注解对注解。<u>**如果**</u> 就是要用，那就接口 `+@CacheNamespace` 并且 对应xml `+<cache-ref namespace="study.lizhuo.mapper.MenMapper"/>`

**其他的相关配置：**

+ 全局配置：cacheEnabled【默认true】
+ statement[ 接口方法 ]配置：userCache 。声明该方法是否使用二级缓存
+ flushCachePolicy：方法配置 ：声明调用该方法后清空缓存。





# SQL重用

指重复使用已经编译过的sql，在此基础上直接设参。他与缓存无关。与statementID也无关，但是其中关于Reuse和Batch之间是有一些差别的。

+ ReuserExcutor：对同一个sql进行连续设参、执行。可查可更新。
+ BatchExecutor：



# Executor执行蓝图

![image-20220720000953185](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220720000953185.png)