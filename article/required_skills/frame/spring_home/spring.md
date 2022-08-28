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

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/001.png)

| **GroupId**         | **ArtifactId**             | **说明**                                           |
| ------------------- | -------------------------- | -------------------------------------------------- |
| org.springframework | [spring-beans]()           | [Beans 支持，包含 Groovy]()                        |
| org.springframework | [spring-aop]()             | [基于代理的AOP支持]()                              |
| org.springframework | [spring-aspects]()         | [基于AspectJ 的切面]()                             |
| org.springframework | [spring-context]()         | [应用上下文运行时，包括调度和远程抽象]()           |
| org.springframework | [spring-context-support]() | [支持将常见的第三方类库集成到 Spring 应用上下文]() |
| org.springframework | [spring-core]()            | [其他模块所依赖的核心模块]()                       |
| org.springframework | [spring-expression]()      | [Spring 表达式语言，SpEL]()                        |
| org.springframework | spring-instrument          | JVM 引导的仪表（监测器）代理                       |
| org.springframework | spring-instrument-tomcat   | Tomcat 的仪表（监测器）代理                        |
| org.springframework | spring-jdbc                | 支持包括数据源设置和 JDBC 访问支持                 |
| org.springframework | spring-jms                 | 支持包括发送/接收JMS消息的助手类                   |
| org.springframework | spring-messaging           | 对消息架构和协议的支持                             |
| org.springframework | spring-orm                 | 对象/关系映射，包括对 JPA 和 Hibernate 的支持      |
| org.springframework | spring-oxm                 | 对象/XML 映射（Object/XML Mapping，OXM）           |
| org.springframework | [spring-test]()            | [单元测试和集成测试支持组件]()                     |
| org.springframework | [spring-tx]()              | [事务基础组件，包括对 DAO 的支持及 JCA 的集成]()   |
| org.springframework | [spring-web]()             | [web支持包，包括客户端及web远程调用]()             |
| org.springframework | [spring-webmvc]()          | [REST web 服务及 web 应用的 MVC 实现]()            |
| org.springframework | spring-webmvc-portlet      | 用于 Portlet 环境的MVC实现                         |
| org.springframework | spring-websocket           | WebSocket 和 SockJS 实现，包括对 STOMP 的支持      |
| org.springframework | [spring-jcl]()             | [Jakarta Commons Logging 日志系统]()               |



# spring核心

## IOC容器

为了规避复杂、传统的构建对象的方式，我们希望有一个管家能够帮助我们维护javaBean的整个生命周期。在构造Bean的时候，他能懂我心意，给他一个眼神（Bean的定义信息），他就能帮我们安排的明明白白，那么这个管家就是IOC容器了，他通过接收管理javaBean来为我们提供省心的服务。不难想象，我们会有一下使用场景：

+ 我有Bean的定义，你来帮我构建一个Bean对象吧
+ 这个Bean对象我不喜欢，你帮我取出来，我来改改
+ 我要用我的Bean，你把它放着就行

可以看出在后期的使用中，我们需要与IOC管家进行交互，他就说想拿什么Bean就去找Beanfactory去，我批准了。那么在一定的理解范围内，我说Beanfactory就是IOC管家应该没错吧。

![image-20220615144919563](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220615144919563.png)



### ApplicationContext和BeanFactory的区别

1. 它们两个都是IOC容器

2. BeanFactory是一个工厂，主要用于根据BeanDefinition来生成Bean，当然也可以它也提供了一些获取Bean的方法。BeanFactroy是延迟注入Bean的，即只有在使用到某个Bean时，才对该Bean进行加载和实例化。<u>【注意】**FactoryBean**</u>是Spring所提供的一种灵话的创建Bean的方式，可以通过实现FactoryBean接口中的getObject( )方法来返回一个Bean对象。 

3. ApplicationContext则在容器启动时，一次性创建了所有的 Bean。ApplicationContext实现了BeanFactory及其他接口，拥有比BeanFactory更多的功能

   + MessageResource == 拥有国际化功能。

   + ListableBeanFactory == 拥有获取beanNames、判断beanName对应beanDefinition对象的存在性、统计BeanDefinition个数等。 

   + HierarchicalBeanFactory == 可以获取父BeanFactory、判断某name是否存在bean对象。

   + EnvironmentCapable，具有获取环境变量的功能，可以获取OS环境变量和JVM环境变量。 

   + ApplicationEventPublisher == 具有事件发布功能
   + ResourceLoader(资源加载器)接口，ApplicationContext可以用来加载多个 Resource。

   + ResourcePatternResolver == 拥有了加载并获取资源的功能（任意URL资源都可以）。 

### 如何将Bean注入IOC容器

![image-20220615153500651](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220615153500651.png)

+ xml配置文件
+ 使用@CompontScan+@Component(@Controller,@Service.@Reposity,@Repository)
+ @Configuration+@Bean
+ @Import
+ 实现@ImportSelector接口，可动态批量的注入Bean

### Bean的加载(实例化)流程

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_22-10-57.png" style="zoom: 67%;" />

1. 程序启动后，由BeanDefinationReader加载配置元信息来源甚广：[.xml,.properties,注解,...]，并将其转化为BeanDefination[说明你要引入哪些bean，它们包含哪些信息]
2. 将BeanDefination注册到BeanDefinationRegistry中
3. 最后由BeanFactoryPostProcessor对BeanDefinationRegistry中的的BeanDefination进行修改。[加入bean的自定义配置，比如datasource的连接信息之类的]

### Bean生命周期

![image-20220615185132728](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220615185132728.png)

1. 查找并加载需要被Spring管理的bean，进行Bean的实例化
2. Bean实例化后会为bean注入依赖
3. 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
4. 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
5. 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。
6. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
7. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
8. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
9. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
10. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。



### Bean作用域

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_22-18-44.png)

## AOP切片

意为面向切面编程，是OOP的延续，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。可用于日志记录、效率检查、事务控制。

SpringAOP的实现依赖于动态代理【JDK动态代理和CGLIB动态代理】，默认使用的是JDK的动态代理方式（基于反射实现），当无法使用JDK动态代理时，AOP就会退而求其次使用CGLIB动态代理（基于继承实现，final类无法进行）

**以打日志为例**，我想在代码段A、B、C的前后插上两条日志来检验执行效率

+ Aspect（切面）：前日志+核心代码段+后日志 组成的一个整体
+ Joint point（连接点）：任意代码段，即所有可以打日志的地方
+ Pointcut（切入点）：声明的约束，用来匹配定位我们想打日志的代码段。
+ Advice（增强）：前日志与后日志
+ Target（目标对象）：目标代码段本身
+ Weaving（织入）：将 前日志+核心代码段+后日志 组合起来的过程，就叫织入

**aop五种通知类型：**

+ 前置通知，**@Before(value = “”)** 在方法执行前通知
+ 后置通知，**@AfterReturning(value = “”)** 在方法正常执行完成进行通知，可以访问到方法的返回值的。
+ 环绕通知，**@Around(value = “”)** 可以将要执行的方法（point.proceed()）进行包裹执行，可以在前后添加需要执行的操作
+ 异常通知，**@AfterThrowing(value = “”)** 在方法出现异常时进行通知，可以访问到异常对象，且可以指定在出现特定异常时在执行通知。

+ 执行后通知，**@After(value = “”)** 在目标方法执行后无论是否发生异常，执行通知,不能访问目标方法的执行的结果。

**<u>*区分@After和@AfterReturning区别，After不论方法是否执正常结束都会通知，而AfterReturning出现异常未正常结束则不会通知。*</u>**

```java
//通知类
@Com..
@Aspect
//表示该类是一个通知类
public class MyAdvice {
    //自己设置一个切点，管理重复代码
	@Pointcut("execution(* com.it.service.*ServiceImpl.*(..))")
	public void pc(){}
	//前置通知
	//指定该方法是前置通知,并制定切入点
	@Before("MyAdvice.pc()")
	public void before(){
		System.out.println("这是前置通知!!");
	}
	//后置通知
	@AfterReturning("execution(* com.it.service.*ServiceImpl.*(..))")
	public void afterReturning(){
		System.out.println("这是后置通知(如果出现异常不会调用)!!");
	}
	//环绕通知
	@Around("execution(* com.it.service.*ServiceImpl.*(..))")
	public Object around(ProceedingJoinPoint pjp) throws Throwable {
		System.out.println("这是环绕通知之前的部分!!");
		Object proceed = pjp.proceed();//调用目标方法
		System.out.println("这是环绕通知之后的部分!!");
		return proceed;
	}
	//异常通知
	@AfterThrowing("execution(* com.it.service.*ServiceImpl.*(..))")
	public void afterException(){
		System.out.println("出事啦!出现异常了!!");
	}
	//后置通知
	@After("execution(* com.it.service.*ServiceImpl.*(..))")
	public void after(){
		System.out.println("这是后置通知(出现异常也会调用)!!");
	}
}
```



## Spring事务

**四大特性：**

+ **原子性：**操作要么同时成功，要么同时失败。
+ **隔离性：**事务「并发」执行时，彼此的操作不能互相干扰。否则就会产生脏读、重复读、幻读的问题。为此InnoDB引擎定义了四种隔离级别
  + read uncommit(读未提交)：可以看到其它事务提交之前的数据  --> <u>脏读</u>
  + read commit (读已提交)：数据在提交后才能被看到  --> <u>不可重复读</u>
  + repeatable read (可重复复读)：指事务执行过程中看到的数据总是跟该事务在启动时看到的数据是一致的。  --> <u>幻读</u>
  + serializable (串行)：同一行记录， “ 写 ” 会加 “ 写锁 ” ， “ 读 ” 会加 “ 读锁 ” 。读写都串行执行。
+ **持久性：**事务一旦提交，对数据库的改变就应该是永久性的。数据的持久化，由redo log实现。
+ **一致性：**保障数据变动前后信息的一致性，这是事务的目的。

**失效条件：**

+ 事务方法的访问权限**不为public**
+ 事务方法被final修饰， 【原因：无法使用代理】
+ 事务方法被this. 调用【没有走代理】，解决方法{ 在当前类中注入当前类的引用，然后用注入后的引用来调用即可 }
+ 事务方法所在的类未被IOC容器管理，如漏写@Service
+ 多线程调用，原因【引发多次的数据库连接，生成多个事务】
+ 数据库表本身不支持事务
+ 项目未开启事务 ，可能出现在老的SSM项目未开启事务支持
+ 使用了不恰当的事务传播
+ try catch 吞掉了异常
+ 自定义的回滚异常与超出的异常不匹配

**xml配置方式：**

```java
<!--  声明式事务管理 -->

<!-- 事务核心管理器,封装了所有事务操作. 依赖于连接池 -->
<bean name="transactionManager"
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 配置事务通知 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!-- 企业中配置CRUD方法一般使用方法名+通配符*的形式配置通知，此时类中的方法名要和配置的方法名一致 -->
        <!-- 以方法为单位,指定方法应用什么事务属性
                isolation：用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔离级别。
                propagation：用于指定事务的传播行为。默认值是REQUIRED，表示一定会有事务，增删改的选择。查询方法可以选择SUPPORTS。
                read-only：用于指定事务是否只读。只有查询方法才能设置为true。默认值是false，表示读写。
                timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，以秒为单位。
                rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值。表示任何异常都回滚。
                no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时事务回滚。没有默认值。表示任何异常都回滚。
             -->
        <tx:method name="save*" isolation="REPEATABLE_READ"
                   propagation="REQUIRED" read-only="false" />
        <tx:method name="persist*" isolation="REPEATABLE_READ"
                   propagation="REQUIRED" read-only="false" />
        <tx:method name="update*" isolation="REPEATABLE_READ"
                   propagation="REQUIRED" read-only="false" />
        <tx:method name="modify*" isolation="REPEATABLE_READ"
                   propagation="REQUIRED" read-only="false" />
        <tx:method name="delete*" isolation="REPEATABLE_READ"
                   propagation="REQUIRED" read-only="false" />
        <tx:method name="remove*" isolation="REPEATABLE_READ"
                   propagation="REQUIRED" read-only="false" />
        <tx:method name="get*" isolation="REPEATABLE_READ"
                   propagation="REQUIRED" read-only="true" />
        <tx:method name="find*" isolation="REPEATABLE_READ"
                   propagation="REQUIRED" read-only="true" />
        <tx:method name="transfer" isolation="REPEATABLE_READ"
                   propagation="REQUIRED" read-only="false" />
    </tx:attributes>
</tx:advice>

<!-- 配置织入 -->
<aop:config>
    <!-- 配置切点表达式 -->
    <aop:pointcut expression="execution(* com.it.service.*ServiceImpl.*(..))" id="txPc" />
    <!-- 配置切面 : 通知+切点 advice-ref:通知的名称 pointcut-ref:切点的名称 -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPc" />
</aop:config>
  
<!-- 注入通知类 -->
<bean name="accountDao" class="com.AccountDaoImpl">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<bean name="accountService" class="com.AccountServiceImpl">
    <property name="accountDao" ref="accountDao"></property>
</bean>
```

**注解配置方式：**

```java
<bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"></property>
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
</bean>

<!-- 事务核心管理器,封装了所有事务操作. 依赖于连接池 -->
<bean name="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 开启spring对注解事务的支持-->
<tx:annotation-driven />

<!-- 注入通知类 -->
<bean name="accountDao" class="com.AccountDaoImpl">
	<property name="dataSource" ref="dataSource"></property>
</bean>

<bean name="accountService" class="com.AccountServiceImpl">
	<property name="accountDao" ref="accountDao"></property>
</bean>
```

**通知类：**

```java
@Transactional(isolation=Isolation.REPEATABLE_READ,propagation=Propagation.REQUIRED,readOnly=true)
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    @Override
    //如果该方法与类名上的配置不同，可以单独在这个方法上配置注解
    @Transactional(isolation= Isolation.REPEATABLE_READ,propagation= Propagation.REQUIRED,readOnly=false)
    public void transfer(Integer from, Integer to, Double money) {
        // 减钱
        accountDao.decreaseMoney(from, money);
        int i = 1 / 0;// 如果发生异常数据（钱）会丢失
        // 加钱
        accountDao.increaseMoney(to, money);
    }

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }
}
```



## 事务传播行为

当存在多个事务方法相互调用的时候，B是用自己的事务还是延用A的事务呢，这需要事务传播机制来调解。

![image-20220615194803791](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220615194803791.png)

|   PROPAGATION_REQUIRED    |  有事务，就加入当前事务。没有事务就开启一个新事务。  |
| :-----------------------: | :--------------------------------------------------: |
|   PROPAGATION_SUPPORTS    |   有事务，就加入当前事务。没有事务就**不用事务**。   |
|   PROPAGATION_MANDATORY   |    有事务，就加入当前事务。没有事务就**报异常**。    |
| PROPAGATION_REQUIRES_NEW  | 开启一个新事务，只用自己的事务。其它事务独立与我无关 |
| PROPAGATION_NOT_SUPPORTED |       总是非事务地执行，并挂起任何存在的事务。       |
|     PROPAGATION_NEVER     |  总是非事务地执行，如果存在一个活动事务，则抛出异常  |
|    PROPAGATION_NESTED     |      如果当前存在事务，则运行在一个嵌套的事务中      |

# 循环依赖

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-15_22-41-42.png)

**循环依赖装配：**

A装B，B递归装A，A又递归装B，。。。

![image-20220615223107441](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220615223107441.png)

