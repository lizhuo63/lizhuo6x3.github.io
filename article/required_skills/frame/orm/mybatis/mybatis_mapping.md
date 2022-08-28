# MyBatis的映射

mybatis是一个ORM框架，M是它的核心，也就是如何将结果集ResultSet中的列转化为JavaBean的对象属性。它的场景复杂，子属性类型多样，包括基本类型、javaBean对象、集合对象、数组等等。为了方便操作，mybatis内置提供了一个强大的反射工具类**MetaObject，**

它支持如下功能：

1. **查找属性**：支持忽略大小写，支持驼峰、支持子属性链式查询，如`a.b.c`;
2. **获取属性：**
   + 可获取对象属性：`a.b`
   + 通过索引访问数组：`ints[0]`
   + 通过键名访问map：`map[men1].username`
3. **设置属性：**
   + 设置子属性数据
   + 支持链式设置子属性时，自动创建子属性对象，【数组和Map除外】



## MetaObject结构&处理流程

![image-20220802124524567](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220802124524567.png)

- **BeanWrapper：** 功能与MeataObject类似，不同点是BeanWrapper只针对单个当前对象属性进行操作，不能操作子属性。
- **MetaClass ：**类的反射功能支持，能获取完整类的属性，包括属性类的属性。
- **Reflector ：**类的反射功能支持，仅支持当前类的属性。

**具体流程**

![image-20220802124808260](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220802124808260.png)

1. 通过MetaObject.getValue("str")进入Mybatis的映射入口。

2. 使用"."将str进行分割，并判断表达式str是否涉及到子属性

   1. 如果有子属性对象，就将他取出，此处是递归操作，直至表达式中再无子属性
   2. 进入BeanWrapper的get()，判断该属性是否含有集合索引
      1. 没有，就说明该属性是基本类型，就通过BeanWrapper.getBeanProperty( )去反射**获取属性值**。
         1. 由 `metaClass.getGetInvoker(prop.getName())` 来反射调用该属性的get方法，最终是由reflector来执行的。
      2. 如果有，就先获取该集合，然后根据索引**获取该集合中指定的值**

   

# ResultMap

映射是指ResultSet的列名与JavaBean属性之间的对应关系，它们的详细信息由ResultMapping进行描述，并被ResultMap封装成一个整体。

![image-20220802145206688](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220802145206688.png)

```xml
<resultMap type="study.lizhuo.entity.Men" id="MenMap">
      <result property="id" column="id"/>
      <result property="username" column="username"/>
</resultMap>
```

![image-20220802145310706](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220802145310706.png)

![image-20220802150218173](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220802150218173.png)

## 几种ResultMap形式

### 自动映射

当列名与属性名相同的情况下会触发自动映射，能够省略<id/> 以及<result/>，但是不支持

<association/> 和<collection/>的省略。

**自动映射条件**

1. 列名和属性名相同（勿略大小写以及驼峰【下划线】）
2. 当前列未手动设置映射，【否则就走手动映射】
3. 属性类别存在TypeHandler
4. 开启autoMapping （默认开启）

### 复合映射

```xml
<resultMap id="OrderDetails" type="Order" extends="BaseResultMap">
  ==================一对一映射=====================
  <association property="product" javaType="Product">
    <id property="id" column="productId"/>
    <result property="productNum" column="productNum"/>
    <result property="productName" column="productName"/>
    <result property="cityName" column="cityName"/>
    <result property="departureTime" column="departureTime"/>
    <result property="productPrice" column="productPrice"/>
    <result property="productDesc" column="productDesc"/>
    <result property="productStatus" column="productStatus"/>
  </association>
  <association property="member" javaType="Member">
    <id property="id" column="memberId"/>
    <result property="name" column="mName"/>
    <result property="nickname" column="mPhoneNum"/>
    <result property="email" column="mEmail"/>
  </association>
  ==================一对多映射======================
  <collection property="travellers" javaType="List" ofType="Traveller">
    <id property="name" column="tName"/>
    <result property="sex" column="tSex"/>
    <result property="phoneNum" column="tPhoneNum"/>
    <result property="credentialsType" column="tCredentialsType"/>
    <result property="credentialsNum" column="tCredentialsNum"/>
    <result property="travellerType" column="tTravellerType"/>
  </collection>
</resultMap>
```

### 嵌套查询

如例：sql[A]->sql[B,C],sql[C]->sql[D],即有三层查询栈：0[A],1[B,C],2[D]

当第0层执行后，或尝试填充属性int,string,B和C，但是发现B,C的属性还未填充，就会转而执行B和C，此后【B的属性已填充完毕，一级缓存有B值了】依次执行C、D，依次去填充D、C，直到C填充完毕，才认为主查询sql[A]执行完毕。

```
<resultMap id="baseMen" type="Men">
  <association property="role" column="rid" select="selectRoleByMen"/>
</resultMap>

rid是从当前查询中获取到的roleID的列名，意为将rid的列数据传给selectRoleByMen。

<select id="selectRoleByMen" resultMap="baseRole">
select * from role where id=#{Id}
<select/>
```

### 外部映射

```xml
<resultMap id="OrderDetails" type="Order" extends="BaseResultMap">
  ==================引用当前resultMap外部的resultMap=====================
  <association property="product" javaType="Product" resultMap="product"/>
</resultMap>

<resultMap id="product" type="Product">
    <id property="id" column="productId"/>
    <result property="productNum" column="productNum"/>
    <result property="productName" column="productName"/>
    <result property="cityName" column="cityName"/>
    <result property="departureTime" column="departureTime"/>
    <result property="productPrice" column="productPrice"/>
    <result property="productDesc" column="productDesc"/>
    <result property="productStatus" column="productStatus"/>
</resultMap>
```



# 循环依赖

Mybatis在i解析对象中的属性时，会先解析填充普通属性，当解析到复合对象时，就会触发对子查询。

1. 如例：sql[A]->sql[B]->sql[A],

+ 在当前查询结束之前，是不会走缓存的



**延迟加载：**在整个主查询【第0层sql查询】结束之后，再往ResultObjiect中填充属性值

延迟加载就是在主查询执行完毕之后，对命中一级缓存（key）的数据（property->resultObject）进行填充

**懒加载：**在只用调用某个属性的get方法的时候，才去加载数据。随用随拿，在一定程度上提高了效率。在Mybatis中，只有手动映射结果集的时候才会存在懒加载。

1. 方法步入queryFromDatabase( )的第一件事，是插入一个缓存占位符，标记存在缓存【虽然只是一个伪符】
2. ResultSetHandler会解析结果集，为A装填属性，但是发现B的属性还未填充，就会率填充B，从而触发子查询
   1. 判断B是否存在缓存值【还未执行查询（没有）】，然后是否开启了懒加载，不然就执行子查询
   2. 自查询再次方法步入queryFromDatabase( )，仍然插入缓存占位符
   3. ResultSetHandler会解析B，为其注入A，发现A已有缓存值【虽然是个伪符】，但任然无法填充【没有具体数据】
   4. 触发延迟加载，将A放入延迟加载的列表中。结束当前层，继续执行第一层的A查询。
   5. 率先判断，当前是否是主查询（查询栈数为0），如果是就遍历执行延迟加载列表中的操作，完成属性赋值。将A注入B，并当即结束查询。

**【总结】：** Mybatis嵌套查询导致的循环依赖直接依靠一级缓存和延迟加载，故即使一级缓存功能jin

```java
// BaseExecutor
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List<E> list;
    // 在查询数据库之初会为当前的MappedStatement创建一个占位符形式的缓存，就是一个标签
*   localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
      localCache.removeObject(key);
    }
```



```java
// 当前查询栈方法执行完后，会由ResultSetHandler处理结果
// 检查目标映射是否是父级映射，然后处理行值，获取子属性的映射值。此时将判断当前的resultMap是否是嵌套的结果集【嵌套查询不再其列】,
private Object getPropertyMappingValue(ResultSet rs, MetaObject metaResultObject, ResultMapping propertyMapping, ResultLoaderMap lazyLoader, String columnPrefix)
      throws SQLException {
    if (propertyMapping.getNestedQueryId() != null) {
      return getNestedQueryMappingValue(rs, metaResultObject, propertyMapping, lazyLoader, columnPrefix);
    }
```



```java
// 获取嵌套查询的映射值
private Object getNestedQueryMappingValue(ResultSet rs, MetaObject metaResultObject, ResultMapping propertyMapping, ResultLoaderMap lazyLoader, String columnPrefix)
      throws SQLException {
     // 为嵌套查询准备素材
    final String nestedQueryId = propertyMapping.getNestedQueryId();
    final String property = propertyMapping.getProperty();
    // 准备MappedStatement
    final MappedStatement nestedQuery = configuration.getMappedStatement(nestedQueryId);
    final Class<?> nestedQueryParameterType = nestedQuery.getParameterMap().getType();
    // 准备sql参数
    final Object nestedQueryParameterObject = prepareParameterForNestedQuery(rs, propertyMapping, nestedQueryParameterType, columnPrefix);
    Object value = null;
    if (nestedQueryParameterObject != null) {
      final BoundSql nestedBoundSql = nestedQuery.getBoundSql(nestedQueryParameterObject);
      final CacheKey key = executor.createCacheKey(nestedQuery, nestedQueryParameterObject, RowBounds.DEFAULT, nestedBoundSql);
      final Class<?> targetType = propertyMapping.getJavaType();
      // 如果命中一级缓存，就执行延迟加载。
      if (executor.isCached(nestedQuery, key)) {
        executor.deferLoad(nestedQuery, metaResultObject, property, key, targetType);
        value = DEFERRED;
      } else {
        final ResultLoader resultLoader = new ResultLoader(configuration, executor, nestedQuery, nestedQueryParameterObject, targetType, key, nestedBoundSql);
        // 如果没有命中一级缓存，就检查是否开启了懒加载
        if (propertyMapping.isLazy()) {
          lazyLoader.addLoader(property, metaResultObject, resultLoader);
          value = DEFERRED;
        } else {
          value = resultLoader.loadResult();
        }
      }
    }
    return value;
  }
```



```java
public void deferLoad(MappedStatement ms, MetaObject resultObject, String property, CacheKey key, Class<?> targetType) {
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    DeferredLoad deferredLoad = new DeferredLoad(resultObject, property, key, localCache, configuration, targetType);
    // 在延迟加载过程中中先判断是否可加载，即是否存在不为占位符的缓存--localCache.getObject(key) != null && localCache.getObject(key) != EXECUTION_PLACEHOLDER;
    if (deferredLoad.canLoad()) {
    	 //执行加载，由MetaObject为子属性设置映射值
      deferredLoad.load();
    } else {
      //如果缓存为占位符，就将它放到
      deferredLoads.add(new DeferredLoad(resultObject, property, key, localCache, configuration, targetType));
    }
  }
```



# 懒加载

通过动态代理，对JavaBean进行代理，在Bean$Proxy中设置了一个handler属性指向ResultLoader（装载器），当我们调用Bean的getXxx( )、toString( )、equal( )时，就会触发懒加载机制，调用handler指向的装载器，进而调用Executor，从而实现Bean的属性填充。

![image-20220807012148221](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220807012148221.png)

**【细节】：**

+ 若在getXxx( )之前调用了setXxx( )，则对应属性的懒加载会被移除，不会覆盖setXxx( )的属性值。

+ 当对象经过序列化和反序列化之后，默认不在支持懒加载。但如果在全局参数中设置了configurationFactory类，而且采用JAVA原生序列化是可以正常执行懒加载的。其原理是将懒加载所需参数以及配置一起进行序列化，反序列化后在通过configurationFactory获取configuration构建执行环境。

  > configurationFactory 是一个包含 getConfiguration 静态方法的类

## 流程

0. Mybatis会为 ResultMap【含有懒加载的子查询】，创建一个增强的代理结果对象，内置了一个MethodHandler和一个ResultLoaderMap【用于存储待加载属性】。

```java
// DefaultResultSetHandler.createResultObject 部分源码
for (ResultMapping propertyMapping : propertyMappings) {
        // 若当前映射含有子查询，且启动了懒加载，便创建一个代理的结果对象
        if (propertyMapping.getNestedQueryId() != null && propertyMapping.isLazy()) {
*         resultObject = configuration.getProxyFactory().createProxy(resultObject, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
          break;
        }
```

```java
// JavassistProxyFactory.createProxy，EnhancedResultObjectProxyImpl实现了MethodHandler
public static Object createProxy(Object target, ResultLoaderMap lazyLoader, Configuration configuration, ObjectFactory objectFactory, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
      final Class<?> type = target.getClass();
      EnhancedResultObjectProxyImpl callback = new EnhancedResultObjectProxyImpl(type, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
*     Object enhanced = crateProxy(type, callback, constructorArgTypes, constructorArgs);
      PropertyCopier.copyBeanProperties(type, target, enhanced);
      return enhanced;
    }
```

![image-20220807092828359](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220807092828359.png)

1. 当调用user.getMen( )获取子对象属性时，会触发懒加载，调用MethodHandler的invoke( )，

   ```java
   // method：bean的原始方法
   // methodProxy:bean的代理后方法
   @Override
       public Object invoke(Object enhanced, Method method, Method methodProxy, Object[] args) throws Throwable {
         final String methodName = method.getName();
         try {
           synchronized (lazyLoader) {
             if (WRITE_REPLACE_METHOD.equals(methodName)) {
               Object original;
               if (constructorArgTypes.isEmpty()) {
                 original = objectFactory.create(type);
               } else {
                 original = objectFactory.create(type, constructorArgTypes, constructorArgs);
               }
               PropertyCopier.copyBeanProperties(type, enhanced, original);
               if (lazyLoader.size() > 0) {
                 return new JavassistSerialStateHolder(original, lazyLoader.getProperties(), objectFactory, constructorArgTypes, constructorArgs);
               } else {
                 return original;
               }
             } else {
               // ResultLoaderMap中是否存在需要加载的属性
               if (lazyLoader.size() > 0 && !FINALIZE_METHOD.equals(methodName)) {
                 // 是否触发所有属性的懒加载
                 if (aggressive || lazyLoadTriggerMethods.contains(methodName)) {
                   lazyLoader.loadAll();
                   //如果有为某属性手动赋值，就取消该属性的懒加载
                 } else if (PropertyNamer.isSetter(methodName)) {
                   final String property = PropertyNamer.methodToProperty(methodName);
                   lazyLoader.remove(property);
                   // 如果调用了get方法
                 } else if (PropertyNamer.isGetter(methodName)) {
                   final String property = PropertyNamer.methodToProperty(methodName);
                   // ResultLoaderMap中是否存在该属性
                   if (lazyLoader.hasLoader(property)) {
                     // 执行该属性的懒加载
                     lazyLoader.load(property);
                   }
                 }
               }
             }
           }
           return methodProxy.invoke(enhanced, args);
         } catch (Throwable t) {
           throw ExceptionUtil.unwrapThrowable(t);
         }
       }
     }
   ```

2. 进入load( )，执行懒加载

   ```java
   public boolean load(String property) throws SQLException {
     // 率先从ResultLoaderMap中移除该属性
       LoadPair pair = loaderMap.remove(property.toUpperCase(Locale.ENGLISH));
       if (pair != null) {
         pair.load();
         return true;
       }
       return false;
     }
   ```

# 联合查询（嵌套映射）

映射是指返回的ResultSet列与Java Bean 属性之间的对应关系。通过ResultMapping进行映射描述，在用ResultMap封装成一个整体。

## 一对一

一行数据会包含多个JavaBean的列，它们会与各自的ResultMap对应，封装成各自的对象，然后根据依赖关系将子对象属性进行填充。

## 一对多

根据查询的多条数据，父Bean的结果均相同，而子Bean的结果存在差异。届时，相同的父Bean将会创建一个父Bean，同时分别创建三个不同的子Bean组成一个集合，并填充至父Bean对象。

### 流程

1. handleRowValuesForNestedResultMap()是嵌套结果集解析入口，在这里会遍历结果集中所有行。并为每一行创建一个RowKey对象。然后调用getRowValue()获取解析结果对象。最后保存至ResultHandler中。

> 注：调用getRowValue前会基于RowKey获取已解析的对象，然后作为partialObject参数发给getRowValue

2. getRowValue()最终会基于当前行生成一个解析好的对象。具体职责包括，1.创建对象、2.填充普通属性和3.填充嵌套属性。在解析嵌套属性时会以递归的方式在调用getRowValue获取子对象。最后一步4.基于RowKey 暂存当前解析对象。

> 如果partialObject参数不为空 只会执行 第3步。因为1、2已经执行过了。**基于此可以实现一对多的解析。**

3. applyNestedResultMappings()解析并填充嵌套结果集映射，遍历所有嵌套映射,然后获取其嵌套ResultMap。接着创建RowKey 去获取暂存区的值。然后调用getRowValue 获取属性对象。最后填充至父对象。

> 如果通过RowKey能获取到属性对象，它还是会去调用getRowsValue，因为有可能属下还存在未解析的属性。



