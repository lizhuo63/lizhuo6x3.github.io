# 实用篇

## 应用构建

创建网络
`docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet` 

### mysql

```
docker run -itd -p 3306:3306 --name=c_mysql8 \
--privileged=true -v /home/lz/public/docker/mysql8/conf:/etc/mysql/conf.d \
--privileged=true -v /home/lz/public/docker/mysql8/logs:/logs \
--privileged=true -v /home/lz/public/docker/mysql8/data:/var/lib/mysql \
--net=mynet --ip=192.168.0.2 \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:8
```

### redis

```
docker run -itd -p 6379:6379 -p 16379:16379 --name c_redis \
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

### portainer

```
docker run -d --name portainerUI -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
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
docker logs rabbitmq
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

