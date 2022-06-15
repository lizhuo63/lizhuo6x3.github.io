

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