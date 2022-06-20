

# mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 命名空间 指向mapper接口的全限定名-->
<mapper namespace="com.lizhuo.mapper.UserMapper"> 
    <resultMap id="baseResultMap" type="com.lizhuo.entity.User">
        <id column="id" javaType="INTEGER" property="id"/>
        <result column="username" javaType="VARCHAR" property="username"/>
        <result column="password" javaType="VARCHAR" property="password"/>
    </resultMap>
    
    <insert id="insertUsers" parameterType="com.lizhuo.entity.User">
        insert into user(username, password) values (#{username},#{password})
    </insert>
</mapper>
```

## 结果集映射

```xml
<!-- 简单结果集，常常对应一个javaBean-->
<resultMap type="Student" id="StudentResult">
    <id property="studId" column="stud_id" />
    <result property="name" column="name" />
    <result property="email" column="email" />
    <result property="phone" column="phone" />
</resultMap>

<!-- 复杂结果集，常常需要多个javaBean来联合构成，通过 extends 实现拼接扩展,用于关联查询wei zhu-->
<resultMap type="Student" id="StudentWithAddressResult" extends="StudentResult">
    <result property="address.addrId" column="addr_id" />
    <result property="address.street" column="street" />
    <result property="address.city" column="city" />
    <result property="address.state" column="state" />
    <result property="address.country" column="country" />
</resultMap>
```



# 传参

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

# 关联查询

## 一对一

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
        association 处理一对一
        封装一对一信息关系的标签
        property  emp类的属性名
        javaType  用哪个类的对象给属性赋值
        -->
        <association property="dept" javaType="dept">
            <id column="deptno" property="deptno"></id>
            <result column="dname" property="dname"></result>
            <result column="loc" property="loc"></result>
        </association>
    </resultMap>

```

## 一对多

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

```java
<resultMap id="deptJoinEmps" type="dept">
        <id column="deptno" property="deptno"></id>
        <result column="dname" property="dname"></result>
        <result column="loc" property="loc"></result>
        <!--处理一对多关系的标签-->
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

## 多对多

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
/
    //组合一个Emp对象作为属性
    private Emp emp;
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
            <association property="emp" javaType="emp">
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























































