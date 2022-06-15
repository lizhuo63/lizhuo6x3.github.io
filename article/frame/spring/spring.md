# Spring生态

+ **Web：**Spring MVC，Spring Web Flux
  + Spring MVC：同步阻塞，基于SpringMVC+Servlet+Tomcat
  + Spring Web Flux：始于Spring5.x，异步非阻塞，基于SpringWebFlux+Reactor+Netty
+ **持久层：**Spring Data JPA、Spring Data Redis、Spring Data MongoDB 
  + Spring Data JPA(Hibernate)与Mybatis基本相似
+ **安全校验：**Spring Security 
+ **工程脚手架：**Spring Boot
+  **微服务：**Spring Cloud 

# Spring架构

Spring是一个轻量级的框架，拥有较为完整的生态

# spring核心

## IOC容器

为了规避复杂、传统的构建对象的方式，我们希望有一个管家能够帮助我们维护javaBean的整个生命周期。在构造Bean的时候，他能懂我心意，给他一个眼神（Bean的定义信息），他就能帮我们安排的明明白白，那么这个管家就是IOC容器了，他通过接收管理javaBean来为我们提供省心的服务。不难想象，我们会有一下使用场景：

+ 我有Bean的定义，你来帮我构建一个Bean对象吧
+ 这个Bean对象我不喜欢，你帮我取出来，我来改改
+ 我要用我的Bean，你把它放着就行

可以看出在后期的使用中，我们需要与IOC管家进行交互，他就说想拿什么Bean就去找Beanfactory去，我批准了。那么在一定的理解范围内，我说Beanfactory就是IOC管家应该没错吧。

![image-20220615144919563](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220615144919563.png)



## ApplicationContext和Beanfactory的区别

BeanFactroy是延迟注入Bean的，即只有在使用到某个Bean时，才对该Bean进行加载和实例化。而ApplicationContext则在容器启动时，一次性创建了所有的 Bean。

他们两个都是IOC容器，ApplicationContext实现了BeanFactory及其他接口，拥有比BeanFactory更多的功能

+ MessageResource 接口对应国际化功能。
+  ResourceLoader(资源加载器)接口，ApplicationContext可以用来加载多个 Resource。
+ EventPublish，具有强大的事件机制

## Bean加载流程

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_22-10-57.png" style="zoom: 67%;" />

1. 程序启动后，由BeanDefinationReader加载配置元信息来源甚广：[.xml,.properties,注解,...]，并将其转化为BeanDefination[说明你要引入哪些bean，它们包含哪些信息]
2. 将BeanDefination注册到BeanDefinationRegistry中
3. 最后由BeanFactoryPostProcessor对BeanDefinationRegistry中的的BeanDefination进行修改。[加入bean的自定义配置，比如datasource的连接信息之类的]

## 生命周期



## 如何将Bean注入IOC容器

![image-20220615153500651](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220615153500651.png)

+ xml配置文件
+ 使用@CompontScan+@Component(@Controller,@Service.@Reposity,@Repository)
+ @Configuration+@Bean
+ @Import
+ 实现@ImportSelector接口，可动态批量的注入Bean

### Bean生命周期



### Bean作用域



## AOP切片



## Spring事务



## 事务传播行为

|   PROPAGATION_REQUIRED    | 有事务，就延用当前事务。没有事务就开启一个新事务。 |
| :-----------------------: | :------------------------------------------------: |
|   PROPAGATION_SUPPORTS    |  有事务，就延用当前事务。没有事务就**不用事务**。  |
|   PROPAGATION_MANDATORY   |   有事务，就延用当前事务。没有事务就**报异常**。   |
| PROPAGATION_REQUIRES_NEW  | 开启一个新事务。如果事务已经存在，就将该事务挂起。 |
| PROPAGATION_NOT_SUPPORTED |      总是非事务地执行，并挂起任何存在的事务。      |
|     PROPAGATION_NEVER     | 总是非事务地执行，如果存在一个活动事务，则抛出异常 |
|    PROPAGATION_NESTED     |  如果一个活动的事务存在，则运行在一个嵌套的事务中  |
