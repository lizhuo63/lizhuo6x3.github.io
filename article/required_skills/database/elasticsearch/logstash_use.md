# 使用流程

## 1. 修改配置 [config/]

### 1.1创建索引模板

```json
{
   "mappings" : {
      "doc" : {
         "properties" : {
            "charge" : {
               "type" : "keyword"
            },
            "description" : {
               "analyzer" : "ik_max_word",
               "search_analyzer" : "ik_smart",
               "type" : "text"
            },
            "end_time" : {
               "format" : "yyyy-MM-dd HH:mm:ss",
               "type" : "date"
            },
            "expires" : {
               "format" : "yyyy-MM-dd HH:mm:ss",
               "type" : "date"
            },
            "grade" : {
               "type" : "keyword"
            },
            "id" : {
               "type" : "keyword"
            },
            "mt" : {
               "type" : "keyword"
            },
            "name" : {
               "analyzer" : "ik_max_word",
               "search_analyzer" : "ik_smart",
               "type" : "text"
            },
            "pic" : {
               "index" : false,
               "type" : "keyword"
            },
            "price" : {
               "type" : "float"
            },
            "price_old" : {
               "type" : "float"
            },
            "pub_time" : {
               "format" : "yyyy-MM-dd HH:mm:ss",
               "type" : "date"
            },
            "qq" : {
               "index" : false,
               "type" : "keyword"
            },
            "st" : {
               "type" : "keyword"
            },
            "start_time" : {
               "format" : "yyyy-MM-dd HH:mm:ss",
               "type" : "date"
            },
            "status" : {
               "type" : "keyword"
            },
            "studymodel" : {
               "type" : "keyword"
            },
            "teachmode" : {
               "type" : "keyword"
            },
            "teachplan" : {
               "analyzer" : "ik_max_word",
               "search_analyzer" : "ik_smart",
               "type" : "text"
            },
            "users" : {
               "index" : false,
               "type" : "text"
            },
            "valid" : {
               "type" : "keyword"
            }
         }
      }
   },
   "template" : "yh_course"
}
```

### 1.2 定制导入导出

```json
# 数据源
input {
  stdin {
  }
  jdbc {
    #此处useSSL=false，可解决CommunicationsException: Communications link failure
    jdbc_connection_string => "jdbc:mysql://localhost:3306/yh_course?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC"
    # 用户名
    jdbc_user => "root"
    # 密码，数字要加引号
    jdbc_password => "123456"
    # mysql驱动文件位置  
    jdbc_driver_library => "H:/workStation/maven/mavenrepository/mysql/mysql-connector-java/5.1.46/mysql-connector-java-5.1.46.jar"
    # MySQL驱动类名称
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    # 允许分页
    jdbc_paging_enabled => "true"
    # 单次采集数量,根据整体数据量调控
    jdbc_page_size => "1000"
    #要执行的sql文件或语句
    #statement_filepath => "/conf/course.sql"
    statement => "select * from course_pub where timestamp > date_add(:sql_last_value,INTERVAL 8 HOUR)"
    # 定时配置
    schedule => "* * * * *"
    # 记录最后的采集参数
    record_last_run => "true"
    # 最后一次采集参数的记录位置
    last_run_metadata_path => "H:/workStation/logstash-7.8.0/config/logstash_metadata"
  }
}


output {
  elasticsearch {
  # ES的ip地址和端口
  hosts => "localhost:9200"
  # hosts => ["localhost:9200","localhost:9202","localhost:9203"]
  # ES索引库名称
  index => "yh_course"
  # 延用数据源的数据id
  document_id => "%{id}"
  # 文档类型，需与模板文件的设置一致
  document_type => "doc"
  # 模板文件的位置
  template => "H:/workStation/logstash-7.8.0/config/yh_course_template.json"
  # 索引名称，需与模板文件的设置一致
  template_name => "yh_course"
  # 允许覆盖，
  template_overwrite => "true"
  }
  stdout {
 #日志输出
  codec => json_lines
  }
}
```

## 2.执行数据同步[bin/]

```bash
logstash.bat -f ../config/mysql.conf
```

