# Mybatis-NoApi

## Controller配置

```java
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Controller"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/controller"))
##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}controller;

import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import javax.annotation.Resource;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表控制层
 *
 * @author lizhuo
 */
@RestController
@RequestMapping("$!tool.firstLowerCase($tableInfo.name)")
public class $!{tableName} {
    /**
     * 服务对象
     */
    @Resource
    private $!{tableInfo.name}Service $!tool.firstLowerCase($tableInfo.name)Service;

}
```

## Entity配置

```java
##引入宏定义
$!{define.vm}

##使用宏定义设置回调（保存位置与文件后缀）
#save("/entity", ".java")

##使用宏定义设置包后缀
#setPackageSuffix("entity")

##使用全局变量实现默认包导入
$!{autoImport.vm}
import lombok.*;
import java.io.Serializable;
#foreach($column in $tableInfo.fullColumn)
#if($column.type.equals("java.util.Date"))
import com.fasterxml.jackson.annotation.JsonFormat;
import org.springframework.format.annotation.DateTimeFormat;
#break
#end
#end
##使用宏定义实现类注释信息
##表注释（宏定义）
#tableComment("表实体类")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class $!{tableInfo.name} implements Serializable {

    private static final long serialVersionUID = $!tool.serial();
    
#foreach($column in $tableInfo.fullColumn)
#if(${column.comment})
${column.comment}
#end
#if($column.type.equals("java.util.Date"))
    @DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss")
    @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss")
#end
    private $!{tool.getClsNameByFullName($column.type)} $!{column.name};
    
#end
}
```

## ServiceImpl配置

```java
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "ServiceImpl"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/service/impl"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}service.impl;

import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import $!{tableInfo.savePackageName}.mapper.$!{tableInfo.name}Mapper;
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表服务实现类
 *
 * @author lizhuo
 */
@Service("$!tool.firstLowerCase($!{tableInfo.name})Service")
public class $!{tableName} implements $!{tableInfo.name}Service {
    @Resource
    private $!{tableInfo.name}Mapper $!tool.firstLowerCase($!{tableInfo.name})Mapper;

}
```

## Service配置

```java
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Service"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/service"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}service;

import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表服务接口
 *
 * @author lizhuo
 */
public interface $!{tableName} {

}
```

## Mapper.java配置

```java
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Mapper"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/mapper"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}mapper;

import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import org.apache.ibatis.annotations.Mapper;
import java.util.List;
/**
* $!{tableInfo.comment}($!{tableInfo.name})表数据库访问层
*
* @author lizhuo
*/
@Mapper
public interface $!{tableName} {
   
   /**
     * 通过Id查询单个
     * @param $!pk.name
     * @return
     */
    $!{tableInfo.name} getOneBy$!{tableInfo.name}Id($!pk.shortType $!pk.name);
    
   /**
     * 通过实体不为空的属性作为筛选条件查询单个
     * 
     * @param $!tool.firstLowerCase($!{tableInfo.name})
     * @return
     */
    $!{tableInfo.name} getOneByEntity($!{tableInfo.name} $!tool.firstLowerCase($!{tableInfo.name}));
    
   /**
     * 通过实体不为空的属性作为筛选条件查询列表
     *
     * @param $!tool.firstLowerCase($!{tableInfo.name})
     * @return
     */
    List<$!{tableInfo.name}> getAllByEntity($!{tableInfo.name} $!tool.firstLowerCase($!{tableInfo.name})); 

   /**
     * 全表查询
     *
     * @return
     */
    List<$!{tableInfo.name}> getAll(); 

   /**
     * 新增单个实体
     * 
     * @param $!tool.firstLowerCase($!{tableInfo.name})
     * @return
     */
    int insertOne($!{tableInfo.name} $!tool.firstLowerCase($!{tableInfo.name}));
  
   /**
     * 批量新增所有列
     * 
     * @param list
     * @return
     */
    int insertBatch(List<$!{tableInfo.name}> list);
    
   /**
     * 通过主键修改实体属性
     * @param $!tool.firstLowerCase($!{tableInfo.name})
     * @return
     */
    int updateBy$!{tableInfo.name}Id($!{tableInfo.name} $!tool.firstLowerCase($!{tableInfo.name}));

   /**
     * 通过主键单个删除
     * @param $!pk.name
     * @return
     */
    int deleteOneBy$!{tableInfo.name}Id($!pk.shortType $!pk.name);
    
   /**
     * 通过主键列表批量删除
     * @param list
     * @return
     */
    int deleteBatchBy$!{tableInfo.name}Ids(List<$!pk.shortType> list);
}
```

## Mapper.xml配置

```xml
##引入mybatis支持
$!mybatisSupport
 
##设置保存名称与保存位置
$!callback.setFileName($tool.append($!{tableInfo.name}, "Mapper.xml"))
$!callback.setSavePath($tool.append($modulePath, "/src/main/resources/mapper"))
 
##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end
 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="$!{tableInfo.savePackageName}.mapper.$!{tableInfo.name}Mapper">
 
    <resultMap type="$!{tableInfo.savePackageName}.entity.$!{tableInfo.name}" id="$!{tableInfo.name}ResultMap"  autoMapping="true">
#foreach($column in $tableInfo.pkColumn)
        <id property="$!column.name" column="$!column.obj.name"/>
#end
#foreach($column in $tableInfo.otherColumn)
        <result property="$!column.name" column="$!column.obj.name"/>       
#end
    </resultMap>
 
    <sql id="allFields">
     (#foreach($column in $tableInfo.fullColumn)$!{column.obj.name}#if($velocityHasNext), #end#end)
    </sql>

    <!--通过Id查询单个-->
    <select id="getOneBy$!{tableInfo.name}Id" resultMap="$!{tableInfo.name}ResultMap" parameterType="$pk.type">
        select *
        from $!tableInfo.obj.name
        where $!pk.obj.name = #{$!pk.name}
    </select>
 
    <!--通过实体不为空的属性作为筛选条件查询单个-->
    <select id="getOneByEntity" resultMap="$!{tableInfo.name}ResultMap" parameterType="$!{tableInfo.savePackageName}.entity.$!{tableInfo.name}">
         select *
        from $!tableInfo.obj.name
        <trim prefix="WHERE" prefixOverrides="AND |OR ">
#foreach($column in $tableInfo.fullColumn)
            <if test="$!column.name != null#if($column.type.equals("java.lang.String")) and $!column.name != ''#end">
               AND $!column.obj.name = #{$!column.name}
            </if>
#end
        </trim>  
    </select>
 
    <!--通过实体不为空的属性作为筛选条件查询列表-->
    <select id="getAllByEntity" resultMap="$!{tableInfo.name}ResultMap" parameterType="$!{tableInfo.savePackageName}.entity.$!{tableInfo.name}">
        select *
        from $!tableInfo.obj.name
        <trim prefix="WHERE" prefixOverrides="AND |OR ">
#foreach($column in $tableInfo.fullColumn)
            <if test="null!=$!column.obj.name and ''!=$!column.obj.name">
               OR $!column.obj.name like concat('%',#{$!column.name},'%')
            </if>
#end
        </trim>   
    </select>

    <!--全表查询-->
    <select id="getAll" resultMap="$!{tableInfo.name}ResultMap">
        select *
        from $!tableInfo.obj.name
    </select>

     <!--新增单个实体-->
    <insert id="insertOne" keyProperty="$!pk.name" useGeneratedKeys="true" parameterType="$!{tableInfo.savePackageName}.entity.$!{tableInfo.name}">
        insert into $!{tableInfo.obj.name}
        <trim prefix="(" suffix=")" suffixOverrides=",">
#foreach($column in  $tableInfo.fullColumn)
          <if test="$!column.name != null">
             $!column.obj.name,
          </if>
#end          
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
#foreach($column in  $tableInfo.fullColumn)
          <if test="$!column.name != null">
             #{$!column.name},
          </if>
#end
        </trim>
    </insert>
       
    <!--批量新增所有列-->
    <insert id="insertBatch" keyProperty="$!pk.name" useGeneratedKeys="true" parameterType="list">
        insert into $!{tableInfo.obj.name}
         (#foreach($column in $tableInfo.fullColumn)$!{column.obj.name}#if($velocityHasNext), #end#end)
        values
        <foreach item="item" collection="list" separator="," open="" close="" index="index">
         (#foreach($column in $tableInfo.fullColumn)#{item.$!{column.name}}#if($velocityHasNext), #end#end)
        </foreach>
    </insert>
    
    <!--通过主键修改实体属性-->
    <update id="updateBy$!{tableInfo.name}Id" parameterType="$!{tableInfo.savePackageName}.entity.$!{tableInfo.name}">
        update $!{tableInfo.obj.name}
        <set>
#foreach($column in $tableInfo.otherColumn)
            <if test="$!column.name != null#if($column.type.equals("java.lang.String")) and $!column.name != ''#end">
                $!column.obj.name = #{$!column.name},
            </if>
#end
        </set>
        where $!pk.obj.name = #{$!pk.name}
    </update>
    
    <!--通过主键单个删除-->
    <delete id="deleteOneBy$!{tableInfo.name}Id" parameterType="$pk.type">
        delete from $!{tableInfo.obj.name} where $!pk.obj.name = #{$!pk.name}
    </delete>
    
    <!--通过主键列表批量删除-->
    <delete id="deleteBatchBy$!{tableInfo.name}Ids" parameterType="list">
        delete from $!{tableInfo.obj.name} where $!pk.obj.name in
        <foreach item="item" collection="list" separator="," open="(" close=")" index="index">
            #{item}
        </foreach>
    </delete>
</mapper>
```





# MyBatisPlus-Api

## Controller配置

```java
##导入宏定义
$!{define.vm}

##设置表后缀（宏定义）
#setTableSuffix("Controller")

##保存文件（宏定义）
#save("/controller", "Controller.java")

##包路径（宏定义）
#setPackageSuffix("controller")

##定义服务名
#set($serviceName = $!tool.append($!tool.firstLowerCase($!tableInfo.name), "Service"))

##定义实体对象名
#set($entityName = $!tool.firstLowerCase($!tableInfo.name))

import $!{tableInfo.savePackageName}.entity.$!tableInfo.name;
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import org.springframework.web.bind.annotation.*;
import io.swagger.annotations.Api;

import javax.annotation.Resource;
import java.io.Serializable;
import java.util.List;

##表注释（宏定义）
@RestController
#tableComment("表控制层")
@Api(tags = "$!{tableName}控制器")
@RequestMapping("$!tool.firstLowerCase($!tableInfo.name)")
public class $!{tableName} {
    /**
     * 服务对象
     */
    @Resource
    private $!{tableInfo.name}Service $!{serviceName};

}
```

## Entity配置

```java
##引入宏定义
$!{define.vm}

##使用宏定义设置回调（保存位置与文件后缀）
#save("/entity", ".java")

##使用宏定义设置包后缀
#setPackageSuffix("entity")

##使用全局变量实现默认包导入
$!{autoImport.vm}
import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.*;
import java.io.Serializable;
#foreach($column in $tableInfo.fullColumn)
#if($column.type.equals("java.util.Date"))
import com.fasterxml.jackson.annotation.JsonFormat;
import org.springframework.format.annotation.DateTimeFormat;
import java.util.Date;
#break
#end
#end
##使用宏定义实现类注释信息
##表注释（宏定义）
#tableComment("表实体类")
@Data
@AllArgsConstructor
@NoArgsConstructor
@TableName("$!{tableInfo.obj.name}")
@ApiModel(value = "$!{tableInfo.name}对象"#if(${tableInfo.comment}), description = "${tableInfo.comment}"#end)
public class $!{tableInfo.name} implements Serializable {

    private static final long serialVersionUID = $!tool.serial();
    
#foreach($column in $tableInfo.pkColumn)
#if(${column.comment})
    @ApiModelProperty(value = "$!{column.comment}")
#end
    @TableId(value="$!{column.name}", type = IdType.AUTO)
    private $!{tool.getClsNameByFullName($column.type)} $!{column.name};
#end

#foreach($column in $tableInfo.otherColumn)
#if(${column.comment})
    @ApiModelProperty(value = "$!{column.comment}")
#end
    @TableField(value="$!{column.name}")
#if($column.type.equals("java.util.Date"))
    @DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss")
    @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss")
#end
    private $!{tool.getClsNameByFullName($column.type)} $!{column.name};

#end
}
```

## Service配置

```java
##导入宏定义
$!{define.vm}

##设置表后缀（宏定义）
#setTableSuffix("Service")

##保存文件（宏定义）
#save("/service", "Service.java")

##包路径（宏定义）
#setPackageSuffix("service")


import $!{tableInfo.savePackageName}.entity.$!tableInfo.name;

##表注释（宏定义）
#tableComment("表服务接口")
public interface $!{tableName} {

}
```

## ServiceImpl配置

```java
##导入宏定义
$!{define.vm}

##设置表后缀（宏定义）
#setTableSuffix("ServiceImpl")

##保存文件（宏定义）
#save("/service/impl", "ServiceImpl.java")

##包路径（宏定义）
#setPackageSuffix("service.impl")

import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import $!{tableInfo.savePackageName}.mapper.$!{tableInfo.name}Mapper;
import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import org.springframework.stereotype.Service;

##表注释（宏定义）
#tableComment("表服务实现类")
@Service("$!tool.firstLowerCase($tableInfo.name)Service")
public class $!{tableName} extends ServiceImpl<$!{tableInfo.name}Mapper, $!{tableInfo.name}> implements $!{tableInfo.name}Service {

    @Resource
    private $!{tableInfo.name}Mapper $!tool.firstLowerCase($!{tableInfo.name})Mapper;

}
```

## Mapper.java配置

```java
##导入宏定义
$!{define.vm}

##设置表后缀（宏定义）
#setTableSuffix("Mapper")

##保存文件（宏定义）
#save("/mapper", "Mapper.java")

##包路径（宏定义）
#setPackageSuffix("mapper")

import java.util.List;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.apache.ibatis.annotations.Mapper;
import $!{tableInfo.savePackageName}.entity.$!tableInfo.name;

##表注释（宏定义）
#tableComment("表数据库访问层")
@Mapper
public interface $!{tableName} extends BaseMapper<$!tableInfo.name> {

}
```

## Mapper.xml配置

```java
##引入mybatis支持
$!{mybatisSupport.vm}

##设置保存名称与保存位置
$!callback.setFileName($tool.append($!{tableInfo.name}, "Mapper.xml"))
$!callback.setSavePath($tool.append($modulePath, "/src/main/resources/mapper"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="$!{tableInfo.savePackageName}.mapper.$!{tableInfo.name}Mapper">

    <resultMap type="$!{tableInfo.savePackageName}.entity.$!{tableInfo.name}" id="$!{tableInfo.name}Map">
#foreach($column in $tableInfo.pkColumn)
        <id property="$!column.name" column="$!column.obj.name"/>
#end
#foreach($column in $tableInfo.otherColumn)
        <result property="$!column.name" column="$!column.obj.name"/>       
#end
    </resultMap>
    
     <sql id="allFields">
     #foreach($column in $tableInfo.fullColumn)$!{column.obj.name}#if($velocityHasNext), #end#end
    </sql>

</mapper>
```



# MyBatis-Plus Cloud

## controller

```java
##导入宏定义
$!{define.vm}

##设置表后缀（宏定义）
#setTableSuffix("Controller")

##保存文件（宏定义）
#save("/controller", "Controller.java")

##包路径（宏定义）
#setPackageSuffix("controller")

##定义服务名
#set($serviceName = $!tool.append($!tool.firstLowerCase($!tableInfo.name), "Service"))

##定义实体对象名
#set($entityName = $!tool.firstLowerCase($!tableInfo.name))

import $!{tableInfo.savePackageName}.entity.$!tableInfo.name;
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import $!{tableInfo.savePackageName}.dto.$!{tableInfo.name}DTO;
import org.springframework.web.bind.annotation.*;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.apache.dubbo.config.annotation.Reference;
import java.util.List;

##表注释（宏定义）
#tableComment("表控制层")
@RestController
@RequestMapping("$!tool.firstLowerCase($!tableInfo.name)")
@Api(tags = "$!{tableInfo.name}控制器")
public class $!{tableName} {

    @Reference
    private $!{tableInfo.name}Service $!{serviceName};

    @GetMapping("{id}")
    @ApiOperation(value = "根据$!{tableInfo.name}Id查询$!{tableInfo.name}")
    public $!{tableInfo.name}DTO get(@PathVariable("id")Integer id) {
        return $!{serviceName}.get(id);
    }
}

```

## entity

```java
##导入宏定义
$!{define.vm}

##保存文件（宏定义）
#save("/entity", ".java")

##包路径（宏定义）
#setPackageSuffix("entity")

##自动导入包（全局变量）
$!autoImport
import lombok.*;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
#foreach($column in $tableInfo.fullColumn)
#if($column.type.equals("java.util.Date"))
import com.fasterxml.jackson.annotation.JsonFormat;
import org.springframework.format.annotation.DateTimeFormat;
import java.util.Date;

import java.io.Serializable;
#break
#end
#end
#tableComment("表实体类")
@Data
@AllArgsConstructor
@NoArgsConstructor
@TableName("$!{tableInfo.obj.name}")
public class $!{tableInfo.name}  implements Serializable {

    private static final long serialVersionUID = $!tool.serial();

#foreach($column in $tableInfo.pkColumn)
#if(${column.comment})
    /*${column.comment}*/
#end
    @TableId(value = "${column.obj.name}",type = IdType.AUTO)
    private $!{tool.getClsNameByFullName($column.type)} ${column.name};
    
#end
#foreach($column in $tableInfo.otherColumn)
#if(${column.comment})
    /*${column.comment}*/
#end
#if($column.type.equals("java.util.Date"))
    @DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss")
    @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss")
#end
    @TableField(value="${column.obj.name}",exist = true)
    private $!{tool.getClsNameByFullName($column.type)} ${column.name};
    
#end
}
```

## mapper.xml

```java
##引入mybatis支持
$!{mybatisSupport.vm}

##设置保存名称与保存位置
$!callback.setFileName($tool.append($!{tableInfo.name}, "Mapper.xml"))
$!callback.setSavePath($tool.append($modulePath, "/src/main/resources/mapper"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="$!{tableInfo.savePackageName}.mapper.$!{tableInfo.name}Mapper">

    <resultMap type="$!{tableInfo.savePackageName}.entity.$!{tableInfo.name}" id="$!{tableInfo.name}Map" autoMapping="true">
    </resultMap>

    <sql id="BaseFields">
     #foreach($column in $tableInfo.fullColumn)$!{column.obj.name}#if($velocityHasNext), #end#end
    </sql>

</mapper>

```

## service

```java
##导入宏定义
$!{define.vm}

##设置表后缀（宏定义）
#setTableSuffix("Service")

##保存文件（宏定义）
#save("/service", "Service.java")

##包路径（宏定义）
#setPackageSuffix("service")

##表注释（宏定义）
#tableComment("表服务接口")
public interface $!{tableName} {

    public $!{tableInfo.name}DTO get(Integer id);

}

```

## serviceImpl

```java
##导入宏定义
$!{define.vm}

##设置表后缀（宏定义）
#setTableSuffix("ServiceImpl")

##保存文件（宏定义）
#save("/serviceImpl", "ServiceImpl.java")

##包路径（宏定义）
#setPackageSuffix("serviceImpl")

import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import $!{tableInfo.savePackageName}.mapper.$!{tableInfo.name}Mapper;
import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import org.apache.dubbo.config.annotation.Service;

##表注释（宏定义）
#tableComment("表Dubbo服务类")
@Service
public class $!{tableName} implements $!{tableInfo.name}Service {

    @Resource
    private $!{tableInfo.name}Mapper $!tool.firstLowerCase($!{tableInfo.name})Mapper;
    
    @Override
    public $!{tableInfo.name}DTO get(Integer id) {

        $!{tableInfo.name} $!tool.firstLowerCase($!{tableInfo.name}) = $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.selectById(id);

        $!{tableInfo.name}DTO $!tool.firstLowerCase($!{tableInfo.name})DTO = new $!{tableInfo.name}DTO();
        //对象对拷
        BeanUtils.copyProperties($!tool.firstLowerCase($!{tableInfo.name}),$!tool.firstLowerCase($!{tableInfo.name})DTO);
        return $!tool.firstLowerCase($!{tableInfo.name})DTO;
    }
}
```

## mapper.java

```java
##导入宏定义
$!{define.vm}

##设置表后缀（宏定义）
#setTableSuffix("Mapper")

##保存文件（宏定义）
#save("/mapper", "Mapper.java")

##包路径（宏定义）
#setPackageSuffix("mapper")

import java.util.List;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import org.springframework.stereotype.Repository;

##表注释（宏定义）
#tableComment("表数据库访问层")
@Repository
public interface $!{tableName} extends BaseMapper<$!tableInfo.name> {

}

```

## dto

```java
##导入宏定义
$!{define.vm}

##保存文件（宏定义）
#save("/dto", "DTO.java")

##包路径（宏定义）
#setPackageSuffix("dto")

##自动导入包（全局变量）
$!autoImport
import lombok.*;
import java.io.Serializable;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
#foreach($column in $tableInfo.fullColumn)
#if($column.type.equals("java.util.Date"))
import com.fasterxml.jackson.annotation.JsonFormat;
import org.springframework.format.annotation.DateTimeFormat;
import java.util.Date;
#break
#end
#end
#tableComment("实体DTO类")
@Data
@AllArgsConstructor
@NoArgsConstructor
@ApiModel(value = "$!{tableInfo.name}对象"#if(${tableInfo.comment}), description = "${tableInfo.name}"#end)
public class $!{tableInfo.name}DTO  implements Serializable {

    private static final long serialVersionUID = $!tool.serial();

#foreach($column in $tableInfo.fullColumn)
    @ApiModelProperty(value = #if($column.comment)"${column.comment}"#else"${column.name}"#end)
#if($column.type.equals("java.util.Date"))
    @DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss")
    @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss")
#end
    private $!{tool.getClsNameByFullName($column.type)} ${column.name};
    
#end
}
```

