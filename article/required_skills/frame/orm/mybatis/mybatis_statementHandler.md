# StatementHandler介绍

它作为JDBC处理器，主要工作是基于JDBC构建Statement并设置参数，然后执行Sql。当缓存未命中时，每调用会话当中的**一次Sql**，都会有唯一与之对应的Statement实例。  

# 工作流程

**核心工作：**声明一个JDBCstatement，然后填充参数，最后执行Sql。

1. 在Executor之前，具体在MapperMethod调用execute( )方法时会先对入参进行解析convertArgsToSqlCommandParam()【具体由ParamNameResolver完成】，然后才是Session委托Executor执行selectList
2. Executor执行query( )，在缓存未命中时，转而查询数据库。在doQuery( )中创建StatementHandler，并为其注入插件。随后执行prepareStatement( )，【该方法先后包括获取数据库连接，预编译和设置sql参数】
   1. 预编译阶段由StatementHandler完成
   2. 设置sql参数分为两步
      1. 由ParameterHandler通过参数名映射参数值。通过boundSql.getParameterMappings()获取参数信息
      2. 通过TypeHandler将参数名映射的参数值打入PreparedStatement
3. 由PreparedStatement执行execute方法，该过程进入JDBC，是对sql的执行。
4. 由ResultSetHandler对结果集进行处理
   1. 由ResultSetHandler将ResultSet中的行数据逐行转化为Java对象【getRowValue( )方法】，并存入ResultContext。
   2. 4.1过程中会由ResultContext控制是否继续读取行数据和javaBean转化，并将javaBean存入**ResultHandler**。



![image-20220731101504924](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220731101504924.png)

【注意：】可在mapper接口方法中使用 `@Options(statementType = StatementType.PREPARED)` 指定 StatementType。默认 PREPARED，如无必要，不建议修改。

# 参数处理

## 参数转换

终极目标：将Java对象转化为sql参数并以Map的形式存储。**处理的时机是在MapperMethod调用invock( )方法时率先对入参进行解析。然后才是Session委托Excutor协调sql的执行。**

![image-20220731150246338](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220731150246338.png)

+ **单个参数，**

  + 如果没有加`@Param`注解，则不转换，直接交给Executor；
  + 如果加了该注解就转化为Map

+ **多个参数，**

  + 默认转化为Map，且键id默认为param1，param2 ...；

  + 如果加了`@Param`注解，并指定了键id，就可自定义sql参数名；

  + 或者通过反射转化成对应的变量名（需要jdk8，并且添加编译参数 `-parameters` 然后clean，否则转化成arg0,arg1...，不建议比较危险）

    ![image-20220731125731164](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220731125731164.png)

    以上arg0 是由反射生成的，暂存到 names 的参数名。最终将有两个参数名映射一个参数值。

```java
// 源码
public static final String GENERIC_NAME_PREFIX = "param";
private final SortedMap<Integer, String> names;

 public ParamNameResolver(Configuration config, Method method) {
    this.useActualParamName = config.isUseActualParamName();
    final Class<?>[] paramTypes = method.getParameterTypes();
    final Annotation[][] paramAnnotations = method.getParameterAnnotations();
    final SortedMap<Integer, String> map = new TreeMap<>();
    int paramCount = paramAnnotations.length;
    // get names from @Param annotations
    for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
      if (isSpecialParameter(paramTypes[paramIndex])) {
        // skip special parameters
        continue;
      }
      String name = null;
      for (Annotation annotation : paramAnnotations[paramIndex]) {
        if (annotation instanceof Param) {
          hasParamAnnotation = true;
          name = ((Param) annotation).value();
          break;
        }
      }
      if (name == null) {
        // @Param was not specified.
        if (useActualParamName) {
          name = getActualParamName(method, paramIndex);
        }
        if (name == null) {
          // use the parameter index as the name ("0", "1", ...)
          // gcode issue #71
          name = String.valueOf(map.size());
        }
      }
      map.put(paramIndex, name);
    }
   // 由上可知names中会优先存储@Param设置的参数名
    names = Collections.unmodifiableSortedMap(map);
  }

public Object getNamedParams(Object[] args) {
  //没有参数就不解析
    final int paramCount = names.size();
    if (args == null || paramCount == 0) {
      return null;
      // 没加@Param注解但只有一个参数，也不解析，直接存储参数名和参数值。
    } else if (!hasParamAnnotation && paramCount == 1) {
      Object value = args[names.firstKey()];
      return wrapToMapIfCollection(value, useActualParamName ? names.get(0) : null);
    } else {
      // 如果有多个参数，就将其转化为Map
      final Map<String, Object> param = new ParamMap<>();
      int i = 0;
      for (Map.Entry<Integer, String> entry : names.entrySet()) {
        // 存入names中的预设的参数名称
        param.put(entry.getValue(), args[entry.getKey()]);
        // add generic param names (param1, param2, ...)
        final String genericParamName = GENERIC_NAME_PREFIX + (i + 1);
        // 如果部分参数没有预设参数名，就存储默认的param1, param2, ...
        if (!names.containsValue(genericParamName)) {
          param.put(genericParamName, args[entry.getKey()]);
        }
        i++;
      }
      return param;
    }
  }
```



## 参数映射和设置

**终极目标：**通过sql参数名【实际就是Mybatis存储的参数列表中参数所映射的Map.key】获取真实的参数值，并将其设置到PreparedStatement中。

+ 当只有一个原始类型参数时，sql参数名可不必与Mapper入参一致，即

  ```
  getUser(int id)  ==> select * from user where id =#{xxx},xxx可任意
  ```

+ **Map类型：**会基于Map.key进行映射

+ **Object：**基于属性名称映射，支持通过"."嵌套访问对象属性

  ```java
  // DefaultParameterHandler核心源码
  public void setParameters(PreparedStatement ps) {
      ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    // 获取参数映射列表 
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
      if (parameterMappings != null) {
        for (int i = 0; i < parameterMappings.size(); i++) {
          ParameterMapping parameterMapping = parameterMappings.get(i);
          if (parameterMapping.getMode() != ParameterMode.OUT) {
            Object value;
            // 获取sql参数的名称
            String propertyName = parameterMapping.getProperty();
            if (boundSql.hasAdditionalParameter(propertyName)) { 
              // 获取sql参数的真实数据
              value = boundSql.getAdditionalParameter(propertyName);
            } else if (parameterObject == null) {
              value = null;
            } else if(typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
              // typeHandlerRegistry内置了基本类型，如果走进来就证明是单参，就直接匹配，映证了单参时的参数名匹配不严格
              value = parameterObject;
            } else {
              // 若参数为java对象，就通过get的方式获取参数值。也是默认的手段
              MetaObject metaObject = configuration.newMetaObject(parameterObject);
              value = metaObject.getValue(propertyName);
            }
            TypeHandler typeHandler = parameterMapping.getTypeHandler();
            // 获取sql的对应数据类型
            JdbcType jdbcType = parameterMapping.getJdbcType();
            if (value == null && jdbcType == null) {
              //打个标记，防止空异常，并且可以提示TypeHandler根据基本类型进行动态匹配
              jdbcType = configuration.getJdbcTypeForNull();
            }
            try {
              typeHandler.setParameter(ps, i + 1, value, jdbcType);
            } catch (TypeException | SQLException e) {
              throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
            }
          }
        }
      }
    }
  ```

  ![image-20220731213533673](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220731213533673.png)

# 结果集封装

**终极目标：**将jdbc返回的ResultSet的行数据，转化成java对象。

1. 读取结果集
2. 遍历结果集当中的行
3. 创建对象
4. 填充属性