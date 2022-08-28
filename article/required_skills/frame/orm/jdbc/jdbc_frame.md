# 介绍

jdbc是java用来操作关系型数据库的一套API，是规范。

![image-20220720182723091](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220720182723091.png)

## 使用流程

```java
    //1.注册驱动
    Class.forName("com.mysql.jdbc.Driver");
    //2.获取连接
    Connection connection = DriverManager.getConnection(url, username, password);
    //3.定义sql
    String sql = "select ...";
    //4.获取sql执行对象
    Statement statement = connection.createStatement();
    //5.执行sql
    ResultSet resultSet = statement.executeQuery(sql);
    //6.释放资源
    statement.close();
    connection.close();
```

# 关键API

## DriverManager 驱动管理器

 **DriverManager** 的核心功能就是 加载驱动，获取连接，日志记录。

![image-20220720184622108](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220720184622108.png)



## Connection 连接/会话

主要用于创建 Statement 、处理事务、获取数据库的相关信息。

![image-20220721091810321](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220721091810321.png)



## Statement

Statement 是sql的可执行对象，用于执行查询和修改、获取sql结果集、以及设计结果集的返回规模。

+ **Statement：**执行静态SQL

  ![image-20220710001136354](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220710001136354.png)

+ **PreparedStatement：**执行预编译SQL，可防SQL注入攻击。采取给" ' "加转义字符来实现。

  批量操作时，PreparedStatement投递的是参数组(需要预编译sql)，而Statementu投递的是若干条sql(预设是投递多种不同的sql)。<u>所以PreparedStatement的效率更优，Statement 的应用场景更广。</u>

+ **CallableStatement：**执行存储过程SQL，可设置出参和读取出参。

![image-20220721092713705](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220721092713705.png)

![image-20220710001907600](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220710001907600.png)



## ResultSet

核心功能是对结果集进行处理，提取对应的列的结果或对其更新。