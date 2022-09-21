# mapper.xml

# #{} 与 ${}区别

**`${}`：底层 Statement**

1. sql与参数拼接在一起，会出现sql注入问题

2. 每次执行sql语句都会编译一次

3. 接收简单数据类型，命名：${value}

4. 接收引用数据类型，命名: ${属性名} 

5. 字符串类型需要加 '${value}'

**`#{}`：底层 PreparedStatement**

1. sql与参数分离，不会出现sql注入问题
2. sql只需要编译一次
3. 接收简单数据类型，命名：#{随便写}
4. 接收引用数据类型，命名：#{属性名} 



# 动态SQL标签

## 增强标签

### trim

```xml
<!-- 多用于标签定制，下述where、set中有记录 -->
<trim prefix="拼接前缀" suffix="拼接" suffixOverrides="判断去除多余后缀" prefixOverrides="判断去除多余前缀"></trim>
```

## 选择匹配

### If 符合匹配（可多选）

```xml
<!-- 匹配所有的 if 标签 -->
<select id="findActiveBlogWithTitleLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

### choose、when、otherwise （单选）

```xml
<!-- 根据传入的属性名匹配对应属性所在的 when 或 otherwise 标签 -->
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

### where

```xml
<!-- 可避免出现 SELECT * FROM BLOG WHERE ,没有判断条件的情况 , 去除有可能多余的 'AND/OR'-->
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <trim prefix="WHERE" prefixOverrides="AND |OR ">
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </trim>
</select>
```

### set

```xml
<!-- 用于匹配更新数据，去除有可能多余的 ',' -->
<update id="updateAuthorIfNecessary">
  update Author
    <trim prefix="SET" suffixOverrides=",">
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </trim>
  where id=#{id}
</update>
```

## 取值遍历

### foreach

```xml
<!-- collection可为任意可迭代对象 -->
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT * FROM POST P WHERE ID in
  <foreach item="item" index="index" collection="array或list或set或map"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```





# 传参

## array

```java
public List<User> findUsersForEacheArray(Integer[] ids);

<select id="findUsersForEacheArray" parameterType="int" resultType="User">
    select * from user where id in
    <foreach collection="array" open="(" close=")" item="id" separator=",">
        #{id}
    </foreach>
</select>
```

## List

```java
List<User> findUsersForEacheList(List<Integer> list);

<select id="findUsersForEacheList" parameterType="list" resultType="User">
    select * from user where id in
    <foreach collection="list" open="(" close=")" item="id" separator=",">
        #{id}
    </foreach>
</select>
```

## 少量参数

```xml
public User selectUser(Integer id,String name);

//按序号传，不方便阅读
<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{0} and dept_id = #{1}
</select>
```

```xml
public User selectUser(@Param("userName") String name, int @Param("deptId") deptId);

//@Param注解传参，还阔以
<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

## 多个参数

```xml
        map.put("id", user.getId());
        map.put("username", user.getUsername());
        map.put("password", user.getPassword());
        map.put("offset", pageRequest.getOffset());
        map.put("pageSize", pageRequest.getPageSize());
List<User> queryAllByLimit(Map<String, Object> map);

//采用map传多参数，对所有用到的变量都要交待。可读性好，就是比较繁琐
    <select id="queryAllByLimit" resultMap="UserMap">
        select
          id, username, password
        from user
        <where>
            <if test="id != null">
                and id = #{id}
            </if>
            <if test="username != null and username != ''">
                and username = #{username}
            </if>
            <if test="password != null and password != ''">
                and password = #{password}
            </if>
        </where>
        limit #{offset}, #{pageSize}
    </select>
```

## 单个Bean

```java
 Integer insertUsers(User user);

//直接用bean字段接收 
<insert id="insertUsers" parameterType="com.lizhuo.entity.User">
	insert into user(username,password) values(#{username},#{password});
</insert>
```

## 多个Bean

```xml
map.put("user", user);
map.put("pageable", pageRequest);
   List<User> queryAllByLimit(Map<String, Object> map);
   
   //使用map传递多个bean，大差不差，就是方便好用
    <select id="queryAllByLimit" resultMap="UserMap">
        select
          id, username, password
        from user
        <where>
            <if test="id != null">
                and id = #{user.id}
            </if>
            <if test="username != null and username != ''">
                and username = #{user.username}
            </if>
            <if test="password != null and password != ''">
                and password = #{user.password}
            </if>
        </where>
        limit #{pageable.offset}, #{pageable.pageSize}
    </select>
```

## JSON对象

```java
controller --> findByJSONObject(@RequestBody JSONObject params){  }

mapper --> List <User> findByJSONObject(JSONObject params);

<select id="findByJSONObject" resultMap="BaseResultMap" parameterType="com.alibaba.fastjson.JSONObject">
  select * from user where username = #{username} and password = #{password}
</select>
```



## 结果集映射

```xml
<!-- 入门级结果集，常常对应一个javaBean-->
<resultMap type="Student" id="StudentResult">
    <id property="studId" column="stud_id" />
    <result property="name" column="name" />
    <result property="email" column="email" />
    <result property="phone" column="phone" />
</resultMap>

<!-- 初级结果集，需要两个javaBean来联合构成，通过extends实现拼接扩展,用于关联查询 -->
<resultMap type="Student" id="StudentWithAddressResult" extends="StudentResult">
    <result property="address.addrId" column="addr_id" />
    <result property="address.street" column="street" />
    <result property="address.city" column="city" />
    <result property="address.state" column="state" />
    <result property="address.country" column="country" />
</resultMap>

<!-- 进阶级结果集，需要两个以上的javaBean且一对一、一对多混合出现，association、collection让结构更清晰-->
 <resultMap id="OrderDetails" type="Order" extends="BaseResultMap">
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



# ？关联查询

## 一对一，立即加载

**实体类**

```
@AllArgsConstructor
@NoArgsConstructor
@Data
public class Emp implements Serializable {
    //...属性省略
    // 组合一个Dept对象作为自己的属性
    private Dept dept;
}
```

**映射文件**

```xml
 <resultMap id="empJoinDept" type="emp">
        <!--设置emp本身的八个属性的映射关系-->
        <id property="empno" column="empno"></id>
        <result property="ename" column="ename"></result>
        <result property="job" column="job"></result>
        <result property="sal" column="sal"></result>
        <result property="hiredate" column="hiredate"></result>
        <result property="mgr" column="mgr"></result>
        <result property="comm" column="comm"></result>
        <result property="deptno" column="deptno"></result>
        <!--
        association 处理一对一封装一对一信息关系的标签
        property: 关联实体的属性名
        javaType: 关联实体的java类型
        -->
        <association property="dept" javaType="com.lizhuo.entity.Dept">
            <id column="deptno" property="deptno"></id>
            <result column="dname" property="dname"></result>
            <result column="loc" property="loc"></result>
        </association>
    </resultMap>

```

## 一对多，延迟加载

**实体类**

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Dept implements Serializable {
//...
    // 组合一个Emp的List集合作为属性
    private List<Emp> empList;
}
```

**映射文件**

```xml
<resultMap id="deptJoinEmps" type="dept">
        <id column="deptno" property="deptno"></id>
        <result column="dname" property="dname"></result>
        <result column="loc" property="loc"></result>
        <!--
        处理一对多关系的标签
        property: 关联实体集合的属性名
        ofType: 关联实体的java类型
        -->
        <collection property="empList" ofType="emp" >
            <!--设置emp本身的八个属性的映射关系-->
            <id property="empno" column="empno"></id>
            <result property="ename" column="ename"></result>
            <result property="job" column="job"></result>
            <result property="sal" column="sal"></result>
            <result property="hiredate" column="hiredate"></result>
            <result property="mgr" column="mgr"></result>
            <result property="comm" column="comm"></result>
            <result property="deptno" column="deptno"></result>
        </collection>
    </resultMap>
```

## 多对多，延迟加载

**实体类**

```java
@NoArgsConstructor
@AllArgsConstructor
@Data
public class Project  implements Serializable {
//...
    // 组合一个ProjectRecord对象集合作为属性
    private List<ProjectRecord> projectRecords;
}
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ProjectRecord implements Serializable {

    //组合一个Emp对象作为属性
    private List<Emp> emps;
}
```

**映射文件**

```xml
  <resultMap id="projectJoinEmps" type="project">
        <id column="pid" property="pid"></id>
        <result column="pname" property="pname"></result>
        <result column="money" property="money"></result>
        <!--一对多 集合属性 collection-->
        <collection property="projectRecords" ofType="projectRecord">
            <id column="empno" property="empno"></id>
            <id column="pid" property="pid"></id>
            <!--一对一 -->
            <association property="emps" javaType="emp">
                <id property="empno" column="empno"></id>
                <result property="ename" column="ename"></result>
                <result property="job" column="job"></result>
                <result property="sal" column="sal"></result>
                <result property="hiredate" column="hiredate"></result>
                <result property="mgr" column="mgr"></result>
                <result property="comm" column="comm"></result>
                <result property="deptno" column="deptno"></result>
            </association>
        </collection>
    </resultMap>
```

# 延迟加载

```xml
<!-- 第一步，开启全局延迟加载配置 -->
<settings>
    <setting name="lazyLoadingEnabled" value="true"/>
</settings>

<!-- 第二步，定制配置目标sql -->
<association></association> 标签
<collection></collection> 标签
	fetchType=""属性
		eager 立即加载
		lazy  延迟加载
```



# 缓存

## 一级缓存

MyBatis一级缓存是：SqlSession级别的缓存，默认开启，不需要手动配置。不同的SqlSession之间的缓存区域是互相不影响的，执行SqlSession的C（增加）U（更新）D（删除）操作，或者调用clearCache()、commit()、close()方法，都会清空缓存

## 二级缓存

二级缓存是mapper映射级别的缓存，虽然它默认开启，但需要在映射文件中配置`<cache/>`标签才能使用，而且要求实体类的必须实现序列化接口。由于二级缓存是跨SqlSession的，所以**关闭一级缓存二级缓存才生效**。多个SqlSession能够操作同一个Mapper映射的sql语句，即多个SqlSession可以共用二级缓存。

```xml
<settings>
    <setting name="cacheEnabled" value="true"/> <!-- mybaits-config.xml中开启全局缓存（默认开启） -->
</settings>

  <!-- ==================================== -->
<mapper namespace="com.it.dao.UserDao">
    <cache /> <!-- 指定缓存 -->

 				 <select id="findById" resultType="com.it.pojo.User" useCache="true" >
      			 select id,name,password FROM t_user where id=#{id}
 				 </select>
</mapper>
```





# 高端玩法

## 分页 pagehelper

* 只有在PageHelper.startPage()方法之后的[第一个查询会有执行分页]()。
* 分页插件[不支持带有“for update”]()的查询语句。
* 分页插件不支持[“嵌套查询”]()，由于嵌套结果方式会导致结果集被折叠，所以无法保证分页结果数量正确。。

```java
//使用代理对象执行方法
PageHelper.startPage(1,2);//设置当前页和每页显示记录数
List<User> users = userDao.findAll();
PageInfo<User> userPageInfo = new PageInfo<>(users);//封装到PageInfo对象中
System.out.println(userPageInfo);
```

![image-20200116145234073](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20200116145234073.png)

## 模糊查询

```xml
 select * from product
        <trim prefix="WHERE" prefixOverrides="AND |OR ">
            <if test="null!=id and ''!=id">
                id like concat('%',#{id},'%')
            </if>
            <if test="productName != null and ''!=productName">
                and productName like concat('%',#{productName},'%')
            </if>
            <if test="cityName != null and ''!=cityName">
                and cityName like concat('%',#{cityName},'%')
            </if>
        </trim>
```

## 批量

### ID删除

```java
 int removeBatcheByProductIds(String[] ids);
```

```xml
<delete id="removeBatcheByProductIds" parameterType="string">
        delete from product where id in
        <foreach item="id" collection="array" open="(" separator="," close=")">
            #{id}
        </foreach>
    </delete>
```

### Bean增修

```java
Integer addBatche(List<User> users);
```

```
<insert id="addBatche" parameterType="java.util.List">
    INSERT INTO user(id, name, password) VALUES
    <foreach collection ="userList" item="user" separator =",">
        (#{user.id}, #{user.name}, #{user.password})
    </foreach >
</insert>
```



### 批量执行方案

#### Java for循环

这种手段性能极差，因为 Mapper 每执行完一个方法都会释放连接，会导致频繁申请和释放资源。

####  foreEach动态标签

它是根据标签里的配置动态生成一条语句，性能很高，但是若数据量很大，则有可能会超出MySQL数据包接收的大小限制。

#### executeBatch执行器

它是将多条sql语句积攒起来，一次发送。性能不错。



### Mybatis分页

Mybatis提供了两种分页方式，逻辑分页(假翻页)和物理分页(真分页)。

#### 逻辑分页

基于内存实现，通过将查询到的所有数据一次性加载到内存，再由逻辑分页对象RowBounds【两个属性：offset和limit】进行筛选，会去掉offset之前和limit之后的数据，然后进行返回。这种方式比较伤内存，万一数据较大就GG了，严格上讲不太安全。

#### 物理分页









