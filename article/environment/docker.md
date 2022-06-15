# 实用篇

## 应用构建

创建网络
`docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet` 

### mysql

```
docker run -itd -p 3306:3306 --name=mysql8 \
--privileged=true -v /home/lz/public/docker/mysql8/conf:/etc/mysql/conf.d \
--privileged=true -v /home/lz/public/docker/mysql8/logs:/logs \
--privileged=true -v /home/lz/public/docker/mysql8/data:/var/lib/mysql \
--net=mynet --ip=192.168.0.2 \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:8
```

### redis

```
docker run -itd -p 6379:6379 -p 16379:16379 --name redis6 \
--privileged=true -v /home/lz/public/docker/redis/data:/data \
--privileged=true -v /home/lz/public/docker/redis/conf/redis.conf:/etc/redis/redis.conf \
--net mynet --ip 192.168.0.3 redis:6 redis-server /etc/redis/redis.conf
```

### redis集群

```
##配置文件模版
port 640${PORT}
bind 0.0.0.0
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf 
cluster-announce-ip 172.18.0.1${PORT}
cluster-announce-port 640${PORT}
cluster-announce-bus-port 1640${PORT}
cluster-node-timeout 5000

##脚本(终端运行)生成9个配置文件
for port in $(seq 1 9); \
do \
  mkdir -p ./node-${port}/conf  \
  && PORT=${port} envsubst <./redis-cluster.tmpl> ./node-${port}/conf/redis.conf \
  && mkdir -p ./node-${port}/data; \
done

##启动容器模版
for port in $(seq 1 9); \
do \
docker run -itd -p 640${port}:640${port} -p 1640${port}:1640${port} --name redis-node${port} \
--privileged=true -v /home/lz/Public/dockers/redis/node/node-${port}/data:/data \
--privileged=true -v /home/lz/Public/dockers/redis/node/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
--net redisnet --ip 172.18.0.1${port} redis:6.0 redis-server /etc/redis/redis.conf; \
done

#创建集群 
redis-cli --cluster create 172.18.0.11:6401 172.18.0.12:6402 172.18.0.13:6403 172.18.0.14:6404 172.18.0.15:6405 172.18.0.16:6406 172.18.0.17:6407 172.18.0.18:6408 172.18.0.19:6409 --cluster-replicas 2
```

### rabbitmq

```
docker run -d --name c_rabbitmq -p 5672:5672 -p 15672:15672 \
--privileged=true -v /home/lz/Public/rabbitmq/data:/var/lib/rabbitmq \
--hostname myRabbit \
--net=mynet --ip=192.168.0.4 \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
rabbitmq:management
```

### elasticsearch

```bash
#1 环境准备
# 1.1 修改系统中允许用户启动的进程可开启的线程数
sudo vim /etc/security/limits.conf
* soft nofile 65536 
*  hard nofile 65536 
*  soft nproc 4096 
*  hard nproc 4096
# 1.2 ElasticSearch需要开辟一个65536字节以上空间的虚拟内存
sudo vim /etc/sysctl.conf
vm.max_map_count=655360
# 1.3 让系统控制权限配置生效
sudo sysctl -p
# 1.4 检查生效情况
sudo sysctl -a|grep vm.max_map_count

#2 拉取elasticsearch
docker pull elasticsearch:7.17.3
# 2.1 创建挂载目录、文件
mkdir -p /public/docker/elasticsearch/es8/data
mkdir -p /public/docker/elasticsearch/es8/config
echo "http.host: 0.0.0.0">>/public/docker/elasticsearch/es8/config/elasticsearch.yml
# 2.2 启动容器
docker run -d --name c_es8 -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
--privileged=true -v /home/lz/public/docker/elasticsearch/es8/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
--privileged=true -v /home/lz/public/docker/elasticsearch/es8/data:/usr/share/elasticsearch/data \
--privileged=true -v /home/lz/public/docker/elasticsearch/es8/plugins:/usr/share/elasticsearch/plugins \
--net mynet --ip 192.168.0.4 elasticsearch:7.17.3
# 验证
http://localhost:9200/

#3 拉取kibana
docker pull kibana:7.17.3
# 3.1 启动容器
docker run -d --name c_kibana --net mynet --ip 192.168.0.5 \
--link c_es8 -p 5601:5601 kibana:7.17.3  #与ES建立关联通过 --link
# 3.2 修改配置
docker exec -it -u root c_kibana /bin/bash
apt-get update
apt-get install vim
vim /config/kibana.yml
改：elasticsearch.hosts: [ "http://192.168.0.4:9200" ]
重启c_kibana
检查：http://localhost:5601/

#4 ik分词器
下载地址：https://github.91chi.fun/https://github.com//medcl/elasticsearch-analysis-ik/releases/download/v7.17.3/elasticsearch-analysis-ik-7.17.3.zip
unzip elasticsearch-analysis-ik-7.17.3.zip -d /home/lz/public/docker/elasticsearch/es8/plugins/ik
重启
```



### portainer

```
docker run -d --name portainerUI -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer-ce
```

### nexus

```
docker run -itd -p 8081:8081 --name=nexus \
--privileged=true -v /home/lz/Public/dockers/nexus/data:/var/nexus-data \
--net=mynet --ip=192.168.0.2 sonatype/nexus3
```



## 服务命令

```
启动：systemctl start docker
状态: systemctl status docker
关闭：systemctl stop docker
重启：systemctl restart docker
开机启动：systemctl enable docker
#无systemctl
service docker start
service docker stop
service docker restart
service docker status

service docker start排障: sudo dockerd --debug
```

## 镜像命令

### 查看

```
##列出本地images
docker images
##含中间映像层
docker images -a
##只显示镜像ID
docker images -q
##含中间映像层
docker images -qa   
##显示镜像摘要信息(DIGEST列)
docker images --digests
##显示镜像完整信息
docker images --no-trunc
##显示指定镜像的历史创建；参数：-H 镜像大小和日期，默认为true；--no-trunc  显示完整的提交记录；-q  仅列出提交记录ID
docker history -H redis
```

### 搜索

```
##搜索仓库MySQL镜像
docker search mysql
## --filter=stars=600：只显示 starts>=600 的镜像
docker search --filter=stars=600 mysql
## --no-trunc 显示镜像完整 DESCRIPTION 描述
docker search --no-trunc mysql
## --automated ：只列出 AUTOMATED=OK 的镜像
docker search  --automated mysql
```

### 下载

```
##下载Redis官方最新镜像，相当于：docker pull redis:latest
docker pull redis
##下载仓库所有Redis镜像
docker pull -a redis
##下载私人仓库镜像
docker pull bitnami/redis
```

### 删除

```
##单个镜像删除，相当于：docker rmi redis:latest
docker rmi redis
##强制删除(针对基于镜像有运行的容器进程)
docker rmi -f redis
##多个镜像删除，不同镜像间以空格间隔
docker rmi -f redis tomcat nginx
##删除本地全部镜像
docker rmi -f $(docker images -q)
```

### 构建

```
##（1）编写dockerfile
cd /docker/dockerfile
vim mycentos
##（2）构建docker镜像
docker build -f /docker/dockerfile/mycentos -t mycentos:1.1 .

#将容器的一切(镜像层和存储层)制成镜像
docker commit -m="提交的描述信息” -a="作者" 容器id 目标镜像名:[TAG]we

tag  为镜像打标签
docker tag 镜像id lizhuo/tomcat:8.1.1
```

## 容器命令

### 查看

```
##查看正在运行的容器
docker ps (-q)
参数：
-a 所有容器
-s 运行容器总文件大小
-l 显示最近创建容器
-n 3 显示最近创建的3个容器
--no-trunc 不截断输出 

##获取镜像redis的元信息
docker inspect redis
##获取正在运行的容器redis的 IP
docker inspect --format='{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis
```

### 启动

```
 docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
##新建并启动容器，参数：
-i  以交互模式运行容器；
-t  为容器重新分配一个伪输入终端；
-d  以守护方式启动容器；
-e	配置环境；
--name  为容器指定一个名称；
-P	随机映射端口；
-p 8080：8080	分配一个指定端口；

docker run -it --name=mycentos1 centos
docker run -id --name=mycentos2 centos
**************************************
docker start redis
docker restart redis
```

### 停止

```
##停止一个运行中的容器
docker stop redis
##杀掉一个运行中的容器
docker kill redis
```

### 进入

```
##进入容器并打开新的终端，可以启动新进程。参数：-i  即使没有附加也保持STDIN 打开；-t  分配一个伪终端
docker exec -it  centos /bin/bash
##以交互模式在容器中执行命令，结果返回到当前终端屏幕
docker exec -it centos ls -l /tmp
##以分离模式在容器中执行命令，程序后台运行，结果不会反馈到当前终端
docker exec -d centos  touch cache.txt 

docker attach centos##进入容器并打开正在执行的终端
```

### 退出

```
##关闭容器并退出
exit
##仅退出容器，不关闭
快捷键：Ctrl + P + Q
```

### 文件

```
 ##将docker容器文件复制到主机
	docker cp 容器:容器内路径 主机文件路径 
##将主机文件复制到docker容器
	docker cp 主机文件路径 容器:容器内路径
```

### 删除

```
##删除一个已停止的容器
docker rm redis
##删除一个运行中的容器
docker rm -f redis
##删除多个容器
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
## -l 移除容器间的网络连接，连接名为 db
docker rm -l db 
## -v 删除容器，并删除容器挂载的数据卷[仅为最后一个使用该卷的容器生效]
docker rm -v rs
```

### 进程

```
##top支持 ps 命令参数，格式：docker top [OPTIONS] CONTAINER [ps OPTIONS]
##列出redis容器中运行进程
docker top redis
##查看所有运行容器的进程信息
for i in  `docker ps |grep Up|awk '{print $1}'`;do echo \ &&docker top $i; done
```

### 日志

```
##查看redis容器日志，默认参数
docker logs redis
##查看redis容器日志，参数：-f  跟踪日志输出；-t   显示时间戳；--tail  仅列出最新N条容器日志；
docker logs -f -t --tail=20 redis
##查看容器redis从2019年05月21日后的最新10条日志。
docker logs --since="2019-05-21" --tail=10 redis
```

## 数据卷

```
-v 容器内路径 	#匿名挂载，不推荐
-v 卷名:容器内路径 	#具名挂载，可用于构建镜像，为镜像新增目录
-v /宿主机路径:/容器内路径 	#指定路径挂载，可用于配置和持久化

#通过 -V容器内路径:ro rw改变读写权限 (针对容器而言)
ro readonly#只读 
rw readwrite#可读可写 
docker run -it --name nginx001 -v /宿主机路径:/容器内路径:ro #容器无法对该文件进行写操作

# 创建一个自定义容器卷
docker volume create edc-nginx-vol
# 查看所有容器卷
docker volume ls
# 查看指定容器卷详情信息
docker volume inspect edc-nginx-vol
# 清理卷， 3步
docker stop edc-nginx
docker rm edc-nginx
docker volume rm edc-nginx-vol
```

## 网络

```
docker network create 
docker network connect 
docker network ls 
docker network rm 
docker network disconnect 
docker network inspect
```

## 构建&发布

### DockerFile

每一个命令就构建了一个新的层

```
由于在构建的过程中docker会采用缓存的机制，如果需要重新构建，不想使用cache需要添加—no-cache。

Dockerfile指令
FROM就是指定基础镜像，是必备的指令，并且必须是第一条指令。有以下格式：
FROM scratch  空白的镜像,不以任何镜像为基础，以后续指令将作为镜像第一层
FROM <image> [AS <name>]
FROM <image>[:<tag>] [AS <name>]
FROM <image>[@<digest>] [AS <name>]

MAINTAINER  指定维护者信息。可以使用LABEL代替
格式为MAINTAINER user_name user_email

LABEL  给镜像添加元数据，设置之后，使用docker inspect来查看。
LABEL <key>=<value> <key>=<value> <key>=<value> ...

RUN  于最上层创建一个新层，并对执行的结果进行提交供dockerfile的后续步骤使用。多个RUN指令可使用&&合并为一个，并且格式要统一。RUN指令是在构建镜像时候执行。
RUN ["executable", "param1", "param2"] 或者 RUN <command>

WORKDIR 指定工作目录的(计入容器厚度当前目录)。如果该目录不存在，则会创建。
格式为： WORKDIR /path/to/workdir

ENV  设置环境变量，供后续指令使用。可以使用docker inspect查的。使用docker run --env <key>=<value>在容器运行时修改环境变量。
ENV <key1>=<value1> <key2>=<value2>... 或者 ENV <key> <value>
[root@master tank]# docker exec tank env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=11c8314f6f5d
NGINX_VERSION=1.17.0
NJS_VERSION=0.3.2
PKG_RELEASE=1~stretch
HOME=/root

ARG  设置环境变量，只在建立image时有效，建立完成后变量就失效消失。对容器无效。该默认值可以在构建命令docker build中用 --build-arg <参数名>=<值> 来覆盖。
格式：ARG <参数名>[=<默认值>]


COPY  将文件从源路径复制添加到容器内部路径。文件来处不限。
格式：COPY [--chown=<user>:<group>] <源路径>... <目标路径> 或者 COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]

ADD  复制文件到容器，会自动下载或者自动解压

VOLUME
定义数据卷，格式为：VOLUME ["<路径1>", "<路径2>"...] 或者 VOLUME <路径>


EXPOSE  指定端口
格式为EXPOSE <端口1> [<端口2>...]。

CMD  CMD指令在每次容器运行的时候执行，如果有多个CMD命令，只有一个最后一个会生效
shell 格式：CMD <命令>
exec 格式：CMD [“可执行文件”, “参数1”, “参数2”…]
在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号。注意，如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式进行执行。即如果CMD echo $HOME，在实际执行中，会将其变更为：CMD [ "sh", "-c", "echo $HOME" ]


ENTRYPOINT  和RUN指令格式一样，但RUN的命令无法追加续写，只能覆盖。在运行时可以通过 docker run 的参数 --entrypoint 来指定。
ENTRYPOINT 的格式和 RUN 指令格式一样，分为 exec 格式和 shell 格式。

ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。ENTRYPOINT 

ENTRYPOINT和CMD同时存在时, docker把CMD的命令拼接到ENTRYPOINT命令之后，即<ENTRYPOINT> "<CMD>", 拼接后的命令才是最终执行的命令。

在一个Dockerfile文件中，如果有多个ENTRYPOINT命令，也只有一个最后一个会生效。而不同点是通过docker run command命令会覆盖CMD的命令，但如果有ENTRYPOINT，则执行的命令不会覆盖ENTRYPOINT，docker run命令中指定的任何参数都会被当做参数传递给ENTRYPOINT。

例如，如果Dockerfile只有 CMD [ "curl", "-s", "https://ip.cn" ]，那么运行 docker run myip -i 会出错，如果是 ENTRYPOINT [ "curl", "-s", "https://ip.cn" ]，运行 docker run myip -i 就会输出Http头部信息，这是因为当存在 ENTRYPOINT 后，CMD 的内容将会作为参数传给 ENTRYPOINT，而这里 -i 就是新的 CMD，因此会作为参数传给 curl，从而达到了我们预期的效果。

有关这两者的区别，可以参考：https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/entrypoint.html 以及 Dockerfile: ENTRYPOINT和CMD的区别

HEALTHCHECK
格式：

HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令
HEALTHCHECK 指令是告诉 Docker 应该如何进行判断容器的状态是否正常，这是 Docker 1.12 引入的新指令。支持下列选项：

--interval=<间隔>：两次健康检查的间隔，默认为 30 秒；
--timeout=<时长>：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
--retries=<次数>：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。
如下的Dockerfile

FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
运行之后就会检测其健康，如下是正常的，healthy:

root@fdm:~/test# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS         NAMES
6cd513fdb549        myweb:v1            "nginx -g 'daemon ..."   19 minutes ago      Up 19 minutes (healthy)   80/tcp        sleepy_booth
.dockerignore
这个不是指令，只是一个文件，用来忽略上下文目录中包含的一些image用不到的文件，它们不会传送到docker daemon。 

```

构建tomcat

```
FROM centos 
MAINTAINER lizhuo<lizhuo6x3@163.com> 

COPY readme.txt /usr/local/readme.txt 
ADD jidk-8u1l-linux-x64.tar.gz /usr/local/ 
ADD apache-tomcat-9.0.22.tar.gz /usr/local/ 

RUN yum -y install vim 

ENV MYPATH /usr/local 
WORKDIR $MYPATH 

ENV JAVA_HOME /usr/local/jdk18.0_11 
ENV CLASSPATH $JAVA_ HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22 
ENV CATALINA_BASH /usr/local/apache-tomcat-90. 22 
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin 

EXPOSE 8080 

CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tail -F /url/local/apache-tomcat-9.0.22/bin/logs/catalina.out 

```

### 发布阿里云

1. 登录阿里云 -> 容器镜像服务 -> 开启个人实例 -> 创建命名空间 -> 创建镜像仓库 -> 进入仓库 -> 按照指南推送镜像 -> 镜像版本中查看镜像

2. ```
    docker login --username=lizhuo6x3 registry.cn-hangzhou.aliyuncs.com                                      
    ```


### 发布docker hub

1. 登录 -> 提交

2. ```
    docker -u lizhuo6x3
    docker push lizhuo6x3/centos:1.0
    ```

# 原理篇

## docker架构图

Docker是一个CS架构的系统，Docker的守护进程运行在主机上，用户通过Socket从客户端访问。Docker 的引擎提供了一组REST API，称为[Docker Remote API](https://docs.docker.com/develop/sdk/)，像 `docker` 命令这样的客户端工具都是通过此API与Docker引擎交互，从而完成各种功能。因此实际上，`docker`的所有命令都是使用的远程调用形式在服务端（Docker 引擎）完成的。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-15_19-11-36.png)

+ **境像( image)**：好比是一个模板,通过这个模板可以创建多个容器( container) 
+ **容器( container)**：独立运行的一个或一组应用进程，是镜像运行时的实体。
+ **仓库( repository)**：存放镜像的地方，仓库分为公有仓库和私有仓库! 
+ **Docker客户端**：Docker是一个典型的C/S架构的应用程序，Docker客户端一般通过Docker command来发起请求，另外，也可以通过Docker提供的一整套RESTful API来发起请求，这种方式更多地被应用在应用程序的代码中。
+ **Docker daemon**：Docker daemon也可理解成Docker Server(Docker Engine)，它实际上是驱动整个Docker功能的核心引擎。就是接收客户端发来的请求，并实现请求所要求的功能，同时针对请求返回相应的结果。

## 容器技术

容器技术，又称容器虚拟化，即一种操作系统虚拟化，是属于轻量级的虚拟化技术。

容器技术主要包括两个内核特性及其他，即容器=cgroup+namespace+rootfs+容器引擎（用户态工具）

+ **Namespace(命名空间)**: 主要做访问隔离。将内核的全局资源做封装，使得每个Namespace都有一份独立的资源，以及容器个体运行互不干扰。目前Linux内核总共实现了6种Namespace：
    + User：隔离用户和组ID。一个进程在Namespace里的用户和组ID与它在host里的ID可以不一样
    + IPC：隔离System V IPC和POSIX消息队列。令相同的标识符在两个Namespace中代表不同的消息队列，使得两个Namespace中的进程不能通过IPC进程通信。
    + Network：隔离网络资源。每个Network Namespace都有自己的网络设备、IP地址、路由表、/proc/net目录、端口号等，是一个虚拟的网络，能使处在不同Network Namespace下的进程可使用同一个端口号而不冲突。
    + Mount：隔离文件系统挂载点。进程系统对文件系统挂载/卸载的动作不会影响到其他Namespace。
    + PID：隔离进程ID。使得不同Namespace里的进程PID号可以是一样的了
    + UTS：隔离主机名和域名。使用主机名在网络上访问某台机器
+ **Cgroup是control group(控制组)**: 主要是做资源控制。将批量容器纳入组，并以组为单位分配资源。用于限制和隔离一组进程对系统资源的使用，主要包括CPU、内存、block I/O和网络带宽。目前各大发行版都默认打开了Cgroup特性。
+ rootfs：文件系统隔离
+ 容器引擎：生命周期控制

## Docker镜像

### 镜像命名

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件。将应用打包成docker镜像,就可以直接跑起来 。好比是一个模板亦或是应用市场的安装包，通过这个模板可以创建多个容器( container) 。以下是dockers image 的表示。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-15_19-22-04.png" style="zoom: 80%;" />

* Remote docker hub：集中存储镜像的Web服务器，默认镜像库即Docker官方镜像库。
* Namespace：表示一个用户或组织
* Repository：表示一个image
* Tag：表示image的版本
* Layer：记录了该版本image的层次构成
* ID：即镜像最上层的layer ID，ID便于脚本处理、操作镜像。

### 镜像结构

docker会对镜像了“分批次的下载”，实际上是记录了对当前层的修改。

原因是docker的镜像是基于Union文件系统的，(UnionFS)最主要的功能是将多个不同位置的目录联合挂载到同一个目录下【有点搭积木的意思】。镜像包含操作系统完整的root文件系统，其体积往往是庞大的，因此利用UnionFS的技术，将其设计为分层存储的架构。所以严格来说，**镜像并非是一个ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现是由一组文件系统组成，或者说，由多层文件系统联合组成**。

所以，**镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层**。比如说，执行删除前一层文件的操作，实际上并没有删除前一层的文件本身，而是仅在当前层标记为前一层文件已删除(即，使‘‘被删层’’对后续层不可见，但本体仍然存在)。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-15_19-30-16.png" style="zoom: 67%;" />

​	Docker image包含着数据及必要的元数据。数据由一层层的image layer组成，元数据则是一些JSON文件，用来描述数据（image layer）之间的关系以及容器的一些配置信息。

graph目录和overlay目录包含本地镜像库中的所有元数据和数据信息。

元数据 (graph目录) 包含json和layersize两个文件，其中json文件包含了必要的层次和配置信息，layersize文件则包含了该层的大小。

## Docker仓库

用来集中存储Docker镜像的服务器

+ 登录到仓库服务：`docker login -u <user name> -p <password> -e <email> <registry domain>`
+ 上传镜像：`docker push localhost:5000/official/ubuntu:14.04`，不写服务器地址则默认上传到官方Docker Hub

## Docker容器

独立运行的一个或一组应用进程，是镜像运行时的实体。提供了一个完整的、隔离的运行环境。

容器是以镜像为基础层，运行时在镜像层上创建一个当前容器的存储层，它是为容器运行时读写而准备的。按照Docker的要求，不直接在存储层读写，采用**数据卷**的方式对宿主机进行读写会有更高的性能和稳定性。存储层依附于容器，而数据卷独立于容器之外。

Docker的镜像是由一系列的只读层组合而来，启动一个容器时，Docker加载镜像的所有只读层，并在最上层加入一个读写层。这个设计可以提高镜像构建、存储和分发的效率，节省了时间和存储空间。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-15_19-37-19.png)

### 数据卷

数据卷（Data Volumes）是宿主机中的一个目录或文件，它以目录的形式呈现给docker容器，是完全独立于容器的生存周期之外的。该目录的主要作用是数据持久化、支持多容器间的数据共享。

当数据卷(宿主机目录)挂在到容器目录后，双方的目录变动会立即同步，以此实现docker的数据持久化（Docker的数据持久化即数据不随着Container的销毁而丢失）。而当一个数据卷被多个容器同时挂载时，就可以实现容器间的数据共享了。

#### 三种挂载方式：

**volumes：**Docker管理宿主机文件系统的一部分，默认位于 /var/lib/docker/volumes 目录中；

**bind mounts：**意为着可以存储在宿主机系统的任意位置；

**tmpfs：**挂载存储在宿主机系统的内存中，而不会写入宿主机的文件系统；

## docker 网络

### 原生docker0

Docker daemon启动时会在主机创建一个Linux网桥（默认为docker0，可通过-b参数手动指定）。每启动一个docker容器, docker就会给该容器分配一个IP，此外只要安装了 docker就会有一个网卡docker0桥接模式,使用的技术是 evth-pair技术! 【 evth-pair就是一对的虛拟的设备接口,他们都是成对出现的,一段连协议,一段连彼此。通过它可以得到虚拟的网络连接，实现宿主与容器】。而容器之间的通信则是通过docker0路由实现的。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-15_21-14-31.png" style="zoom:67%;" />

docker0这个默认的虚拟网卡有很大的缺陷，**不支持直接通过“域名”(容器名)访问。**即 ping mycentos1 失败，破法：

`docker run -it --name=mycentos2 --link mycentos1  mycentos:1.0`

如此可实现mycentos2对mycentos1的基于容器名的连接，然而这是单向的，从mycentos1则无法基于容器名来连接mycentos2，原因是**--link的本质是在该容器的hosts中配置了新的ip,域名的映射,而这是单向的**。

### CNM

Docker使用了CNM ( Container Network Model)容器网络模型对Docker的网络进行了抽象。CNM定义了构建容器虚拟化网络的模型，同时还提供了可以用于开发多种网络驱动的标准化接口和组件。

**CNM的3个核心组件：**

+ 沙盒：一个隔离的网络运行环境，保存了容器网络栈配置，包括对网络接口、路由表和DNS配置的管理。
+ Endpoint：Endpoint将沙盒加入指定网络。
+ 网络：网络包括一组能互相通信的Endpoint，实现可以是Linux bridge、vlan等

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-15_21-30-43.png" style="zoom:67%;" />

#### CNM的五种驱动

+ **bridge：**Docker默认的容器网络驱动。通过一对veth pair将Container连接到docker0网桥上，veth pair就是一对的虚拟设备接口，它是成对出现的。一端连着协议栈，一端彼此相连着。由Docker为容器动态分配IP及配置路由、防火墙规则等。docker0网桥等同于交换机，为连在其上的设备转发数据帧，但docker0有情景限制。
+ **host：**容器与主机共享同一Network Namespace，共享同一套网络协议栈、路由表及iptables规则等。容器与主机看到的是相同的网络视图。
+ **null：**容器内网络配置为空，需要用户手动为容器配置网络接口及路由等。
+ remote：Docker网络插件的实现。Remote driver可以通过HTTP RESTful API对接第三方的网络方案。
+ overlay：Docker原生的跨主机多子网网络方案。

#### 基本网络配置

+ none：不为容器配置任何网络功能。需要以`--net=none`参数启动容器

+ container：与另一个运行中的容器共享Network Namespace，举例：

    1. 首先以默认网络配置（bridge模式）启动一个容器，设置hostname为dockerNet，dns为8.8.4.4。

        ` docker run -h dockerNet --dns 8.8.4.4 -tid ubuntu:latest bash`

    2. `docker exec -ti d25864df1a3b bash` 进入容器，`ip addr show`查看网络

    3. `docker run --net=container:d25864df1a3b -ti ubuntu:latest bash`然后以--net=container：d25864df1a3b方式启动另一个容器，`ip addr show`查看网络

    4. 发现网络配置继承了容器d25864df1a3b

+ host：与主机共享Root Network Namespace，容器有完整的权限可以操纵主机的协议栈、路由表和防火墙等，是不安全的。

    + `docker run -ti --net=host ubuntu:latest bash`;`ip addr show`

+ bridge：Docker设计的NAT网络模型。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-16_21-32-20.png" style="zoom:67%;" />

+ overlay：Docker原生的跨主机多子网模型

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-16_21-34-12.png" style="zoom:67%;" />

每创建一个网络，Docker会在主机上创建一个单独的沙盒（实质上是一个Network Namespace）。在沙盒中，Docker会创建名为br0的网桥，并在网桥上增加一个vxlan接口并占用一个vxlan ID,[vxlan隧道的ID范围为256~1000]。当添加一个容器到某一个网络上时，Docker会创建一对veth网卡，一端连接到此网络相关沙盒内的br0网桥上，另一端放入容器的沙盒内，并设置br0的IP地址作为容器内路由默认的网关地址，从而实现容器加入网络的目的。

#### docker 网络参数配置

**Docker daemon部分**

```
# 指定Docker daemon使用的网桥，默认为"docker0"。若设置-b="none"，则禁用Docker的网络功能
-b, --bridge=
# 指定docker0网桥的IP，注意--bip不能与-b同时使用
--bip= 
# 设置容器默认的IPv4网关
--default-gateway= 
# 设置容器默认的IPv6网关
--default-gateway-v6=
# 设置容器内的DNS服务器地址
--dns-search=[]
# 设置容器内的search domain，即域名解析时默认添加的域名后缀
--fixed-cidr=
# 与--fixed-cidr相同，bridge模式下默认分配的IPv6子网地址
--fixed-cidr-v6=
# 指定Docker client与Docker daemon通信的socket地址，可以是tcp地址、unix socket地址
-H, --host=[] 
	#Docker daemon同时监听10.110.48.32:10000及本机/var/run/docker.sock文件
	docker daemon -H tcp://10.110.48.32:10000 -H unix:///var/run/docker.sock
# 允许/禁止容器间通信，禁用icc依赖iptables规则，若--icc=false，则必须--iptables=true
--icc=true
# 容器暴露端口时默认绑定的主机IP，默认为0.0.0.0
--ip=0.0.0.0
# 使能IP转发功能，为true则向主机/proc/sys/net/ipv4/ip_forward写入1
--ip-forward=true
# 使能IP地址变形功能（NAT）,只有--iptables=true才可生效
--ip-masq=true
# 使能iptables。若设置为false，则无法向iptables表格添加规则
--iptables=true
# 使能IPv6网络功能
--ipv6=false
# 设置容器网络MTU（最大传输单元）
--mtu=0
```

**Docker client部分**

```
# 在容器内的/etc/hosts文件内增加一行主机对IP的映射
--add-host=[]
# 设置容器的DNS服务器
--dns=[]
# 设置容器的DNS搜索域
--dns-search=[]
# 暴露容器的端口，而不映射到主机端口
--expose=[]
# 设置容器的主机名
-h, --hostname=
# 链接到另一个容器，在本容器中可以通过容器ID或者容器名访问对方
--link=[]
# 设置容器的mac地址
--mac-address= 
# 设置容器的网络运行模式，当前支持四种模式：bridge、none、host、container
--net=bridge
# 将容器所有暴露出的端口映射到主机随机端口
-P, --publish-all=false 
# 将容器一段范围内的端口映射主机指定的端口
-p, --publish=[]
```

**创建网卡**

```
##创建网络
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet 
##查看网络
docker inspect mynet 

##重新启动容器
docker run -id -P --name=mycentos1 --net=mynet mycentos:1.0
docker run -id -P --name=mycentos2 --net=mynet mycentos:1.0
## 测试“域名”访问
❯ docker exec -it mycentos1 ping mycentos2
PING mycentos2 (192.168.0.3) 56(84) bytes of data.
64 bytes from mycentos2.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.109 ms
64 bytes from mycentos2.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.108 ms
```

​	可见，自定义的网络完善了docker0的缺陷，支持对“域名”的g访问。

**实现不同网络下容器的联通**

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-16_21-41-34.png" style="zoom:67%;" />

无法直接联通容器，也不能直接联通网络，而是应该**将容器连接到多个网络上**。

```
~  docker run -id -P --name=mycentos3  mycentos:1.0     
~  docker run -id -P --name=mycentos4  mycentos:1.0
❯ docker network connect mynet mycentos3 ##将mycentos3接入mynet
❯ docker inspect mynet
~  docker exec -it mycentos3 ping mycentos1	##联通成功 
~  docker exec -it mycentos4 ping mycentos1	##联通失败
```

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-05-16_21-40-10.png)



## 容器卷

​		由于容器的数据都保存在存储读写层，而存储层依附于容器，所以当容器被删除时，其数据也会被一同删除，十分的不安全。

​		数据卷就是通过将容器的目录系统挂载到宿主文件系统的指定目录下，借助实时同步，使容器的数据与挂载点的数据实现双向同步。和系统安装时挂载点的分配是一样的。简单来讲，卷就是目录或文件，由Docker daemon挂载到容器中，因此不属于联合文件系统，卷中的数据在容器被删除后仍然可以访问。

Docker提供了两种管理数据的方式：数据卷和数据卷容器。主要有两个应用。一，构建镜像时为容器新建目录。二，启动容器时，为指定目录创建同步。

### 常用命令

```
# 将主机目录挂载为数据卷
docker run -d -v /host/data:/data --name busyboxtest busybox
# 以只读的方式挂载一个数据卷
docker run -ti -v /host/data:/data:ro --name busyboxtest busybox
```

### 数据卷容器

​		如果要在容器之间共享一些需要永久存储的数据，可以创建一个数据卷容器，然后使用该容器进行数据共享。使用数据卷容器存储的数据不会轻易丢失，即便删除db1、db2容器甚至是初始化该数据卷的dbdata容器，该数据卷也不会被删除。只有在删除最后一个使用该数据卷的容器时显式地指定docker rm–v$CONTAINER才会删除该数据卷。

```
#1 先创建一个数据卷容器，该容器中并不运行任何应用：
docker create -v /dbdata --name dbdata training/postgres /bin/true
#2 然后启动服务，使用--volumes-from将数据卷挂载进来，再启动多个容器，各容器之间就可通过dbdata数据卷共享数据了
docker run -d --volumes-from dbdata --name db1 training/postgres
docker run -d --volumes-from dbdata --name db2 training/postgres
docker run -d --name db3 --volumes-from db1 training/postgres

```

只支持本地数据卷，但允许使用者以第三方插件的形式，提供对分布式存储的支持。

## Dockerfile

核心功能就是在基础镜像之上，构建出预期的镜像。

### 基本语法

**由于在构建的过程中docker会采用缓存的机制，如果需要重新构建，不想使用cache需要添加—no-cache。**

**FROM**：就是指定基础镜像，是必备的指令，并且必须是第一条指令。有以下格式：

* `FROM scratch`  空白的镜像,不以任何镜像为基础，以后续指令将作为镜像第一层
* `FROM <image> [AS <name>]`
* `FROM <image>[:<tag>] [AS <name>]`
* `FROM <image>[@<digest>] [AS <name>]`

**MAINTAINER**：指定维护者信息。可以使用LABEL代替

+ `MAINTAINER user_name user_email`

**LABEL**：  给镜像添加元数据，设置之后，使用docker inspect来查看。

+ `LABEL <key>=<value> <key>=<value> <key>=<value> ...`

**EXPOSE**：将容器中的端口号暴露出来

+ `EXPOSE <端口1> [<端口2>...]`

**RUN**：  于最上层创建一个新层，并对执行的结果进行提交供dockerfile的后续步骤使用。多个RUN指令可使用&&合并为一个，并且格式要统一。RUN指令是在构建镜像时候执行。

+ `RUN ["executable", "param1", "param2"] `或者` RUN <command>`

**CMD**：指定启动容器时执行的命令，每个Dockerfile只能有一条CMD指令。如果指定了多条CMD指令，只有最后一条会被执行；如果用户启动容器时指定了运行的命令，则会覆盖掉CMD指定的命令。

+ shell 格式：`CMD <命令>`  ;实际的命令会被包装为 sh -c 的参数的形式进行执行,即如果CMD echo $HOME，在实际执行中，会将其变更为：CMD [ "sh", "-c", "echo $HOME" ]
+ exec 格式：`CMD [“可执行文件”, “参数1”, “参数2”…] ` ;在解析时会被解析为 JSON 数组，因此一定要使用双引号

**ENTRYPOINT**：指定容器启动程序及参数，和RUN指令格式一样，但RUN的命令无法追加续写，只能覆盖。当指定多个时，只有最后一个有效。在运行时可以通过 docker run 的参数 --entrypoint 来指定。

+ shell 格式：`ENTRYPOINT command param1 param2`
+ exec 格式：`ENTRYPOINT["executable"，"param1"，"param2"]`

​		<u>ENTRYPOINT和CMD同时存在时, docker把CMD的命令拼接到ENTRYPOINT命令之后，即<ENTRYPOINT> "<CMD>", 拼接后的命令才是最终执行的命令。docker run command命令会覆盖CMD的命令，却不会覆盖ENTRYPOINT，docker run命令中指定的任何参数都会被当做参数传递给ENTRYPOINT。</u>[详细见ENTRYPOINT和CMD的区别](https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/entrypoint.html)

​		例如，如果Dockerfile只有 CMD [ "curl", "-s", "https://ip.cn" ]，那么运行 docker run myip -i 会出错，如果是 ENTRYPOINT [ "curl", "-s", "https://ip.cn" ]，运行 docker run myip -i 就会输出Http头部信息，这是因为当存在 ENTRYPOINT 后，CMD 的内容将会作为参数传给 ENTRYPOINT，而这里 -i 就是新的 CMD，因此会作为参数传给 curl，从而达到了我们预期的效果。

**VOLUME**：创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库或需要永久保存的数据。如果和host共享目录，Dockerfile中必须先创建一个挂载点，然后在启动容器的时候通过“docker run –v$HOSTPATH：$CONTAINERPATH”来挂载，其中CONTAINERPATH就是创建的挂载点。

+ 格式为：`VOLUME ["<路径1>", "<路径2>"...]` 或者 `VOLUME <路径>`

**ENV**： 设置环境变量，供后续指令使用。可以使用docker inspect查看，使用docker run --env <key>=<value>在容器运行时修改环境变量。

+ `ENV <key1>=<value1> <key2>=<value2>... `或者 `ENV <key> <value>`

**ARG**：设置环境变量，只在建立image时有效，建立完成后变量就失效消失。对容器无效。该默认值可以在构建命令docker build中用 --build-arg <参数名>=<值> 来覆盖。

+ 格式：`ARG <参数名>[=<默认值>]`

**ADD**： 复制文件到容器，会自动下载或者自动解压

+ 格式为：`ADD <src> <dest>`

**COPY**：复制本地主机的<src>（为Dockerfile所在目录的相对路径）到容器中的<dest>。当使用本地目录为源目录时，推荐使用COPY。

+ 格式：`COPY [--chown=<user>:<group>] <源路径>... <目标路径> `或者 `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

**WORKDIR**：指定当前工作目录的(切换到指定目录)。如果该目录不存在，则会创建。

+ 格式为： `WORKDIR /path/to/workdir`

**USER**：指定当前用户；注意`USER`只是切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。

+ 格式：`USER <用户名>[:<用户组>]`

**ONBUILD**：指定以当前镜像为基础镜像去构建下一级镜像时执行的命令

+  格式：`ONBUILD <其它指令>`

**HEALTHCHECK**：检查容器健康状况

+ HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令，告诉Docker应该如何判断容器是否正常
+ HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令
    + --interval=<间隔>：两次健康检查的间隔，默认为 30 秒；
    + --timeout=<时长>：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
    + --retries=<次数>：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。

例如：

```dockerfile
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
  
运行之后就会检测其健康，如下是正常的，healthy:
root@fdm:~/test# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS         NAMES
6cd513fdb549        myweb:v1            "nginx -g 'daemon ..."   19 minutes ago      Up 19 minutes (healthy)   80/tcp        sleepy_booth
```

**.dockerignore**：这个不是指令，只是一个文件，用来忽略上下文目录中包含的一些image用不到的文件，它们不会传送到docker daemon。 

查看容器的元数据：docker exec mysql5 env

### 发布

**阿里云**：

1. 登录阿里云 -> 容器镜像服务 -> 开启个人实例 -> 创建命名空间 -> 创建镜像仓库 -> 进入仓库 -> 按照指南推送镜像 -> 镜像版本中查看镜像

2. ```
    docker login --username=lizhuo6x3 registry.cn-hangzhou.aliyuncs.com                                      
    ```


**docker hub**

1. 登录 -> 提交

2. ```
    docker -u lizhuo6x3
    docker push lizhuo6x3/centos:1.0
    ```

    

