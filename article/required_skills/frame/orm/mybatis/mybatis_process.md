# MyBatis执行流程

```java
InputStream resource = StartDemo.class.getResourceAsStream("/mybatis-config.xml");
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(resource);
        Configuration configuration = sessionFactory.getConfiguration();
        SqlSession sqlSession = sessionFactory.openSession();
        Class<?> user = configuration.getTypeAliasRegistry().getTypeAliases().get("user");
        log.info(user.getName()); // study.lizhuo.entity.User
        sqlSession.close();
        resource.close();
```

## 配置解析阶段[获取SqlSession]

![image-20220710145226426](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220710145226426.png)

1. 根据配置文件【"/mybatis-config.xml"】获取输入流。它会进一步根据<mappers>标签读取Mapper.xml的配置信息。
2. **SqlSessionFactoryBuilder** 解析流数据并生成配置类，用于构建 **SqlSessionFactory** 。配置的解析是由 **XMLConfigBuilder** 完成的。Session是MyBatis的交互门面，而 SqlSessionFactory 是用于创建  **Session** 的。该过程 SqlSessionFactoryBuilder 用到了建造者模式，它实现通过流中的元数据动态的构建SqlSessionFactory。SqlSessionFactory 用于开启会话以及获取 **Configuration**<u>【**Configuration包含MyBatis的所有配置**】</u>，它采取了工厂模式，提供了包含4种入参的8种构造。{autoCommit、connection、transactionIsolationLevel、executorType}
3. SqlSessionFactory 开启 SqlSession，此过程中会创建Transaction和CachingExecutor【内部包装有具体的执行器类型】，并获取**连接**。


```java
    // Mybatis源码,Mybatis根据它获取SqlSession对象
    private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      final Environment environment = configuration.getEnvironment();
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit); //设置事务
      final Executor executor = configuration.newExecutor(tx, execType); //设置执行器,此时为CachingExecutor
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```

## MapperMethod代理阶段

1. 进入执行入口，由MapperMethod匹配方法类型

2. 由CachingExecutor执行query( )方法

   1. 判断是否存在二级缓存。如果存在就进入缓存链，获取缓存结果。
   2. 如果二级缓存中没有，就进入BaseExecutor**执行doQUery( ),**
      1. 在doQuery( )中创建StatementHandler，并为其注入插件。随后执行prepareStatement( )，【该方法先后包括获取数据库连接，预编译和设置sql参数】
         1. 预编译阶段由StatementHandler完成
         2. 设置sql参数分为两步
            1. 由ParameterHandler通过参数名映射参数值。通过boundSql.getParameterMappings()获取参数信息
            2. 通过TypeHandler将参数名映射的参数值打入PreparedStatement
      2. 由PreparedStatement执行execute方法，该过程进入JDBC，是对sql的底层执行。
      3. 由ResultSetHandler对结果集进行处理
         1. 由ResultSetHandler将ResultSet中的行数据逐行转化为Java对象【getRowValue( )方法】，并存入ResultContext。
         2. 4.1过程中会由ResultContext控制是否继续读取行数据和javaBean转化，并将javaBean存入**ResultHandler**。

   

## sql执行阶段

![image-20220710130801871](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220710130801871.png)

#### **SqlSession：**

![image-20220717161904913](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220717161904913.png)

#### **Executor：**

它是在开启会话时创建的，用于服务整个会话，所以它不可能是针对个别 MapperMethod 的设计，它调控的一定是会话期间所有sql的共性执行方案。例如，事务方式，缓存配置，

**创建流程：**

```java
 		// mybatis安照如下顺序创建Executor，当开启二级缓存时，会直接对BaseExecutor进行包装
		Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
```

![image-20220717223251585](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220717223251585.png)

##### SimpleExecutor

最基本的执行器，每一次执行都必须走完完整的流程。执行流程为：编译sql -> 设置参数 -> (执行sql) -> 接收结果。

##### ReuseExecutor

ReuseExecutor内部有一个 statementMap 来缓存statement，执行sql之前会判断，statement缓存中有没有改sql，如果命中就直接取并设参执行。它会将编译后的sql缓存下来，避免了重复编译。执行流程为：编译sql -> **设置参数 -> (执行sql) -> 接收结果** -> 设置参数 -> (执行sql) -> 接收结果。

##### BatchExecutor

它只**<u>对连续出现的修改sql起效果</u>**，如果传入的是查询sql，那么功效将变成 SimpleExecutor。它的执行流程为：编译sql -> **设置参数 -> 设置参数 -> (执行sql) **-> 接收结果。【**注意：**BatchExecutor必须执行doFlushStatements( )，该方法实现了对statementList的遍历，executeBatch( )批量执行和结果收集。不然数据库数据将无更新】

除此之外，对于update语句，要达到statement共用的要求，还必须要将相同的sql写的挨到一块，一但有不同的sql插进来就会再次执行编译。

```java
			//currentSql为上一次执行的sql，只有当相同的sql连续调用时，才能实现批量提交
			if (sql.equals(currentSql) && ms.equals(currentStatement)) {
      int last = statementList.size() - 1;
      stmt = statementList.get(last);
      applyTransactionTimeout(stmt);
      handler.parameterize(stmt);// fix Issues 322
      BatchResult batchResult = batchResultList.get(last);
      batchResult.addParameterObject(parameterObject);
    } else {
      Connection connection = getConnection(ms.getStatementLog());
      stmt = handler.prepare(connection, transaction.getTimeout());
      handler.parameterize(stmt);    // fix Issues 322
      currentSql = sql;
      currentStatement = ms;
      statementList.add(stmt);
      batchResultList.add(new BatchResult(ms, sql, parameterObject));
    }
```



##### 总结

+ 横向来比，SimpleExecutor的效率最差；BatchExecutor属于是打包sql语句，一次发送，搞不好会触碰到Mysql 4MB的数据包大小限制，但效率最佳；ReuseExecutor还阔以吧。
+ 纵向来看，在使用geMapper()调用MapperMethod时，形同的sql[包括参数]将只执行一次【全过程：编译、设参、执行。】，是一级缓存起效果了

```
 @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, args);
      } else {
        return cachedInvoker(method).invoke(proxy, method, args, sqlSession);
      }
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
  }
```



#### **StatementHandler：**

一个StatementHandler只处理一个方法