# 安装

## ElasticSearch

```bash
1 tar -zxvf elasticsearch-7.17.5-linux-x86_64.tar.gz -C /opt

2 vim /opt/elasticsearch-7.17.5/config/elasticsearch.yml
  cluster.name: my-application
  node.name: node-1
  network.host: 0.0.0.0
  http.port: 9200
  cluster.initial_master_nodes: ["node-1"]

3 sudo vim elasticsearch-env
  # 配置使用ES内置的jdk(避免与本机环境冲突)
  export ES_JAVA_HOME=/opt/elasticsearch-7.17.5/jdk
  export CLASSPATH=$ES_JAVA_HOME/lib/
  export PATH=$PATH:$ES_JAVA_HOME/bin

4 [root@boy opt]# 
		sudo useradd es
		sudo passwd 
		chown -R es:es elasticsearch-7.17.5/

5 vim /etc/security/limits.conf
  es soft nofile 65536
  es hard nofile 65536

6 vim /etc/security/limits.d/20-nproc.conf
  *          soft    nproc     4096
  root       soft    nproc     unlimited
  es         soft    nofile    65536
  es         hard    nofile    65536
  *          hard    nproc     4096
  
7 vim /etc/sysctl.conf
  vm.max_map_count=655360
  
8 [root@boy bin]# sysctl -p

# 启动
9 [root@boy bin]# su bata
  [bata@boy bin]$ ./elasticsearch
```

**配置：**

```yaml
# 集群的名字
cluster.name: my-application
# 节点的名称
node.name: master
# 数据存放的位置
path.data: D:/elasticsearch-datas/to/data
# 日志存放的位置
path.logs: D:/elasticsearch-datas/to/logs
# 节点绑定的IP
network.host: 192.168.43.240
# 设置端口号
http.port: 9200
# 集群中所有的节点的ip地址
discovery.seed_hosts: ["192.168.43.240", "192.168.223.139"]
# 初始状态下集群中主节点的 node.name
cluster.initial_master_nodes: ["master"]
```

## kibana

```bash
1 tar -zxvf kibana-7.17.5-linux-x86_64.tar.gz -C /opt

2 vim /opt/kibana-7.17.5-linux-x86_64/config/kibana.yml
  server.port: 5601
  server.host: 0.0.0.0
  server.name: "kibana-application"
  elasticsearch.hosts: ["http://127.0.0.1:9200"]
  elasticsearch.requestTimeout: 99999 
```

## 分词器

```bash
1 安装ik,切记安装预构建版本，release
2 指令
mkdir /opt/elasticsearch-7.17.5/plugins/analysis-ik
cp elasticsearch-analysis-ik-7.16.0.zip /opt/elasticsearch-7.17.5/plugins/analysis-ik/
cd /opt/elasticsearch-7.17.5/plugins/analysis-ik
sudo unzip elasticsearch-analysis-ik-7.16.0.zip
sudo cp -R config/* /opt/elasticsearch-7.17.5/config/

3 重启es
```

## LogStash

**配置：**

```properties
input {
  file {
    path => "D:/logstash-datas/movies.csv"
    start_position => "beginning"
    sincedb_path => "D:/elasticsearch/logstash-7.4.2/db_path.log"
  }
}
filter {
  csv {
    separator => ","
    columns => ["id","content","genre"]
  }

  mutate {
    split => { "genre" => "|" }
    remove_field => ["path", "host","@timestamp","message"]
  }

  mutate {

    split => ["content", "("]
    add_field => { "title" => "%{[content][0]}"}
    add_field => { "year" => "%{[content][1]}"}
  }

  mutate {
    convert => {
      "year" => "integer"
    }
    strip => ["title"]
    remove_field => ["path", "host","@timestamp","message","content"]
  }

}
output {
   elasticsearch {
     hosts => "http://localhost:9200"
     index => "movies"
     document_id => "%{id}"
   }
  stdout {}
}
```



# ElasticSearch须知

## 倒排索引

说白了就是关键字与关联数据的检索顺序的区别。当用户发起关键词查询时，搜索引擎会扫描索引库中的所有文档，找出所有包含关键词的文档，这样依次**从文档中去查找是否含有关键词**的方法叫做正向索引。而反之，直接根据关键词去检索数据的唯一标识并获取整个数据的方法就叫做倒排索引。



## 数据格式

**类比理解：** ElasticSearch是面向文档类型的数据库，旨在通过内容关键字检索出关联的所有文档内容，若是采用Mysql的设计，

![image-20220724010421762](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220724010421762.png)

|   DBMS   |           Elasticsearch            |
| :------: | :--------------------------------: |
| database |               Index                |
|  table   |  type(在7.0之后type为固定值_doc)   |
|   Row    |              Document              |
|  Column  |               Field                |
|  Schema  |              Mapping               |
|   SQL    | DSL(Descriptor Structure Language) |

+ **索引(index)** 
  ElasticSearch存储数据的地方，可以理解成关系型数据库中的数据库概念。 
+ **映射(mapping)** 
  mapping定义了每个字段的类型、字段所使用的分词器等。相当于关系型数据库中的表结构。 
+ **文档(document)** 
  Elasticsearch中的最小数据弹元，常以json格式显示。一个document相当于关系型数据库中的一行数据。 

## 数据类型支持

### 简单类型

#### 字符串

+ text：可分词，但不支持聚合
+ keyword：不可分词，支持聚合

#### 数值

+ long 带符号的64位整数。
+  integer 带符号的32位整数。
+ short 带符号的16位整数。
+ byte 带符号的8位整数。
+ double 双精度64位IEEE754浮点数。
+ float 单精度32位EEE754浮点数。 
+ half float 半精度16位1EEE754浮点数，限制为有限。 
+ scaled_float 支持有限浮点数。 

#### 数组  [ ]

#### 对象  { }

# 索引操作

## curl方式

+ **添加索引** 
  PUT http://ip:端口/索引名称 

+ **查询索引** 
  GET http://ip:端口/索引名称 

+ **删除索引** 
  DELETE http://ip:端口/索引名称 

+ **关闭索引** 

  POST http://ip:端口/索引名称/__close 

+ **打开索引** 
  POST http://ip:端口/索引名称/_open 

## kibana方式

+ **添加索引**
  PUT 索引名称 
+ **查询索引**
  GET 索引名称 
+ **删除索引**
  DELETE 索引名称 
+ **关闭索引**
  POST 索引名称/_close
+ **打开索引**
  POST 索引名称/_open 

# 映射操作

+ 添加映射

  ```json
  put men/_mapping
  {
    "properties":{
      "name":{
        "type":"keyword"
      },
      "age":{
        "type":"integer"
      }
    }
  }
  ```

+ 创建索引并添加映射

  ```json
  put men
  {
    "mapping": {
      "properties": {
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
  ```

+ 查询映射

  ```
   GET men/_mapping
  ```

+ 添加映射字段

  ```json
  put men/_mapping
  {
    "properties":{
      "sex":{
        "type":"keyword"
      }
    }
  }
  ```

# 文档操作

## 添加

+ 添加文档【指定id】

  ```json
  PUT/POST men/_doc/1
  {
    "name":"tb",
    "age":18,
    "sex":"男"
  }
  ```

+ 添加文档【不指定id】

  ```json
  POST men/_doc
  {
    "name":"tb",
    "age":18,
    "sex":"男"
  }
  ```

+ 添加文档【id重复即报错】

  ```json
  PUT user/_create/1 
  {
    "name":"tb",
    "age":18,
    "sex":"男"
  }
  ```

+ 批量添加(可以指定ID，也可以不指定ID)

  ```
  POST men/_bulk
  {"index":{"_id": 3}}
  {"name":"tb","age":8,"sex":"男"}
  {"index":{}}
  {"name":"nb","age":4,"sex":"男"}
  ```


## 删改

+ 修改文档

  ```javascript
  POST men/_update/1
  {
    "doc": {
      "name":"lv",
      "age":20,
      "sex":"女"
    }
  }
  ```

+ 删除文档

  ```json
  DELETE men/_doc/1
  ```

## 查询

+ 查询id文档

  ```json
  GET men/_doc/1
  ```

+ 查询所有文档

  ```json
  GET men/_search
  ```

+ 批量查询

  ```json
  GET _mget
  {
    "docs": [
      {"_index":"men", "_id":"1"},
      {"_index":"men", "_id":"2"}
    ]
  }
  ```

+ 分页查询，对查询结果分页

  ```json
  GET men/_search
  {
    "from": 0,
    "size": 20
  }
  ```

## 泛查询（URI查询）

查询条件中的空格代表 " || "，

+ 不指定字段 `GET men/_search?q=18`

+ 指定字段 `GET men/_search?q=age:18` 或者`GET men/_search?q=18&df=age`

+ 指定字段和查找范围 `GET men/_search?q="男"&df=sex&from=1&size=8`
+ **多条件、判断查询** 
  + `GET men/_search?q=age:12 16`，查age为12或者16。空格代表”||“
  + `GET movies/_search?q=title:(+Beautiful Mind)`，或者【+可省略，-代表排除】
  + `GET men/_search?q=age:12 AND sex="男"`，查age为12或者16。空格代表”||“
  + `GET men/_search?q=age:8 AND sex:="男"`，查age为8的男性。并且必须 ”AND“。
  + `GET movies/_search?q=title:(Mind AND Beautiful)`，**并且**
  + `GET movies/_search?q=year:(>=2012 AND <2018)`，**范围**
  + `GET movies/_search?q=year:{2015 TO 2017]`，范围【需以 ] 结尾】，{ 不含，[ 包含
  + `GET movies/_search?q=title:Mi?d`，**正则表达式**

##  RequestBody复杂查询

+ 范围+排序+分页

  ```json
  GET movies/_search
  {
    "sort": [
      {
        "year": {
          "order": "desc"
        }
      }
    ],
    "query": {
      "range": {
        "year": {
          "gte": 2017,
          "lte": 2018
        }
      }
    },
    "from": 30,
    "size": 5
  }
  ```

+ 多字段匹配+排序

  ```json
  GET movies/_search
  {
    "sort": [
      {
        "year": {
          "order": "desc"
        }
      }
    ],
    # query中只能有一个条件 [不能有注释]
    "query": {
      "match": {
        "title": "Beautiful Mind"
      }
    }
  }
  ```

+ **多条件查询必须使用bool**

  ```json
  GET movies/_search
  {
    "query": {
      "bool": {
        "must": [
          {
            "range": {
              "year": {
                "gte": 2017,
                "lte": 2018
              }
            }
          },
          {
            "match": {
              "title": "Beautiful Mind"
            }
          }
        ]
      }
    }
  }
  ```

+ 只查询部分列

  ```javascript
  GET movies/_search
  {
    "_source": ["title", "year"]
  }
  ```

+ 整句匹配

  ```json
  GET movies/_search
  {
    "query": {
      "match_phrase": {
        "title": "Beautiful Mind"
      }
    }
  }
  ```

+  多字段匹配【自带排序策略】

  ```javascript
  GET movies/_search
  {
    "query": {
      "multi_match": {
        "query": "beautiful mind Romance",
        "fields": ["title", "genre"],
        "type": "best_fields"
      }
    }
  }
  ```

  其中type的值有三个:

  - most_fields：在多字段中匹配的越多排名越靠前   
  - best_fields: 能完全匹配的文档，排名越靠前。 
  - cross_fields: 查询越分散，排名越靠前。

+ **query_string:**

  ```json
  GET movies/_search
  {
    "query": {
      "query_string": {
        "fields": ["title", "genre"],
        "query": "Beautiful Mind",
        "default_operator": "AND"
      }
    }
  }
  ```

+ `term`实现精准匹配

  ```javascript
  GET movies/_search
  {
    "query": {
      "term": {
        "title.keyword": {
          "value": "Captain Marvel"
        }
      }
    }
  }
  ```

+  **推荐搜索，三个推荐策略**

  ```javascript
  GET movies/_search
  {
    "suggest": {
      "title-suggest": {
        "text": "minx",
        "term": {
          "field": "title",
          "suggest_mode": "missing"
        }
      }
    }
  }	
  
  ```

   **"suggest_mode":**

  +  "missing"，未匹配时，触发推荐
  + "popular"，自动触发，热点推荐
  + "always"，

  ## 自动补全

  Elasticsearch的自动补全功能是基于 `suggest` 来实现的，需要提前定义好需要进行补全搜索字段的mapping信息为 `completion` (mapping一旦创建好后是不能修改的)。



