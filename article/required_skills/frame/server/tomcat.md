# tomcat结构图

+ Server：代表着整个服务器

  + 但此时整个服务都耦合在一起了，后续将其分为请求监听和请求处理两块。

+ Service：是一个Connector容器。用于高效绑定成对的请求和响应，即将同批次的任务交由一个Service来管理

+ Connector：负责开启Socket并监听请求，返回数据

+ Container：负责处理请求，后被细分为Engine、Host、Context、Wrapper。

+ **Engine：List<Host>** 代表Web的应用容器，是Servlet的引擎，虚拟主机的直接容器

+ **Host：List<Context>** 部署服务对应的域名（虚拟主机），是Web应用的直接容器

+ **Context：List<Wrapper=Servlet类>** 表示一个独立的服务应用，是Servlet类的直接容器。**本应是Servlet实例的直接容器，但是对于一个Servlet类来讲若是出现了多实例场景，就不便于管理，且结构也不明朗，就出现了Wrapper结构**

+ **Wrapper：List<Servlet实例>** 代表一个Servlet类，是Servlet实例的直接容器

  ![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20211202093312956.png)



<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-24_09-56-28.png" style="zoom:70%;" />

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-24_01-45-00.png" style="zoom:70%;" />

+ **Conector升级改造：**
  + **ProtocoHandler：**是一个协议处理器，根据不同协议和I/O方式提供不同实现支持
  + **AbstractEndpoint：**用于在Connector启动之后，启动线程监听服务器端口，在收到请求后调用Processor
  + **Processor：**用于在收到请求后，根据URL映射容器Container



+ **Service升级改造：**
  + **Mapper：**用于维护容器的映射列表，以及匹配容器
  + **MapperListener：**用于监听容器状态，并实时更新容器的映射信息，它随service一同启动
  + **CoyoteAdapter：**默认的Tomcat适配器连接方案，是实现Connector和Container解耦的核心



+ **Executor并发升级：**
  + AbstractEndpoint接收到请求后会启动一组线程来监控Socket，接收到请求后会生成请求对象转交由Executor实现并发处理



+ **环境配置升级改造：**
  + **Catalina：**它提供了Shell程序，负责启动停用服务。可解析server.xml来创建各种组件
  + **Bootstrap：**它是服务器启动的入口，负责创建Catalina实例。那么我为什么不直接通过Catalina来启动呢？==> Bootstrap可以直接依赖JRE运行，能很方便地为Tomcat创建共享类加载器，用于构造Catalina实例以及整个服务器。



# Tomcat工作流程

## 启动流程

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-24_11-13-07.png)

## 请求流程

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-24_11-15-51.png)

## 类加载器 (流程)

Tomcat的类加载要考虑到如下场景

+ **隔离性：**不同的应用通常有不同的依赖，必须在加载时将他们隔离开，否则会因为Jar包覆盖导致启动异常
+ **综合考虑：**每个应用都应该有自己的一套加载器，不仅可以屏蔽其他应用**（安全）**，也能确定应用自身的最小加载范围**（性能）**，此外对自身应用的维护不会干扰到其他应用**（灵活）**
+ **共用性：**对一些共用的资源，应当指定若干公用类加载器加载

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-24_13-24-19.png)

+ **Common类加载器：**加载的Web服务器和各个应用均可见的Jar包，是公共资源

+ **Catalina类加载器：**加载一些仅对Web服务器可见的类，对Web应用不可见

+ **Share类加载器：**加载对所有Web应用均可见的类，是Web应用的共用类

  

  **Web应用类加载器的加载顺序：**

  1. 先从缓存中加载
  2. 未命中，就从JVM的Bootstrap中加载
  3. 未命中，就从当前类加载器加载（先/classes再/lib）
  4. 未命中，就从父类加载器加载



# Servlet容器 Catalina

## Digester

​		一个将XML转化为Java对象的工具，Catalina需要通过Digester来解析server.xml配置文件并据此来创建相关组件。它是支持异步的。

## 解析Server

1. 创建Server实例。默认实现为`org.apache.catalina.core.StandardServer`,也可通过className指定自己的实现
2. 为Server实例设置全局命名上下文`Server/GlobalNamingResources`
3. 为Server添加生命周期的监听器
4. 构造Service实例
5. 为Service添加生命周期的监听器
6. 为Service添加Executor
7. 为Service添加Connector
8. 为Connector添加虚拟主机`SSLHostConfig`配置
9. 为Connector添加生命周期的监听器
10. 为Connector添加升级协议`Server/Service/Connector/UpgradeProtocal`
11. 添加子元素解析规则，具体是将各级子节点封装为RuleSet，包括GlobalNamingResources、Engine、Host、Context、Cluster。

### 解析Engine

1. 创建Engine实例
2. 为Engine添加集群配置
3. 为Engine添加生命周期监听器
4. 为Engine添加安全配置以及拦截器Valve

### 解析Host

1. 创建Host实例
2. 为Host添加集群配置，【集群配置既能在Engine也能在Host】
3. 为Host添加生命周期监听器
4. 为Host添加安全配置以及拦截器Valve【默认为AccessLogValve，用于记录日志】

### 解析Context

1. 创建Context实例，会伴随添加ContextConfig(一个监听器)，用于详细配置Context，如解析web.xml
2. 为Context添加生命周期监听器
3. 为Context指定类加载器
4. 为Context添加会话管理器
5. 为Context添加初始化参数，可在context.xml中进行配置，以实现多web应用的复用
6. 为Context添加安全配置和资源配置，包括PreResources、JarResources、PostResources。
7. 为Context添加资源链接
8. 为Context添加Valve拦截器
9. 为Context添加守护资源配置。`WatchedResource`用于监听资源变更实现热加载，`WrapperLifecycle`用于为Wrapper添加生命监听器，``WarpperListener`用于为Wrapper添加容器监听器，
10. 为Context添加Cookie处理器

​		Servlet容器的**核心功能**就是部署应用和将请求映射到对应的Servlet