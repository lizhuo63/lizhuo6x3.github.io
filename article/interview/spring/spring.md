# Spring

### Bean作用域

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_22-18-44.png)

###  Bean生命周期

1. 实例化Bean
2. 依赖注入
3. 检查该Bean实现的相关Aware接口，并设置对应的属性
4. 检查该Bean是否自定义 init-method，有就调用
5. 检查该Bean是否实现了BeanPostProcessor 接口，有将会调用postProcessAfterIn
   itialization(Object obj, String s)方法
6. Bean初始化成功，投入使用
7. 检查该Bean是否实现了DisposableBean这个接口，有就调用destroy()方法
8. 最后检查这个Bean是否自定义了destroy-method属性，有就调用自配置的销毁方法

### 循环依赖

**示例：**A-->B,C；B-->A；C-->A，设先创建A

**种类：**

- ①构造器的循环依赖：这种依赖spring是处理不了的，直接抛出BeanCurrentlylnCreationException异常。
- ②单例模式下的setter循环依赖：通过“三级缓存”处理循环依赖，能处理。
- ③非单例循环依赖：无法处理。原型(Prototype)的场景是不支持循环依赖的，通常会走到`AbstractBeanFactory类中下面的判断，抛出异常。`

**三个缓存：**

+ **singletonObjects(一级缓存)** 俗称“单例池”，用于存储完整的单例bean。
+ **earlySingletonObjects(二级缓存) **保存没有经过完整生命周期的bean，属性值为null，同时保证该bean即使不完整也是单例的。它主要用于保证B、C能够拿到同一个A。
+ **singletonFactories(三级缓存) **映射创建Bean的原始工厂。在bean创建之初就不管有没有循环依赖，都会根据beanName，beanDefinition和当前的普通bean生成一个**lamda表达式**，存入三级缓存。**它是打破循环的关键。** 该表达式也是AOP的执行入口，核心作用是动态判断生成一个普通对象或是代理对象。

当 Spring 为某个 Bean 填充属性时，首先会寻找需要注入对象的名称，然后依次执行 getSingleton() 方法得到所需注入的对象，而获取对象的过程就是依次从一级缓存、二级缓存中获、三级缓存中获取，如果三级缓存中也没有，那么就会去执行 doCreateBean() 方法创建这个 Bean。

**Spring解决流程：**

1. 创建A

   0. alreadyCreated("a")，记录下来

   1. 创建a普通对象 --> [此时A中的b、c均为null]根据当前A的残缺的bean、beanName、beanDefinition生成一个lambda表达式，存入三级缓存。
   2. 注入属性，B、C
      1. 去单例池中找b、c ，没有就创建b、c
         1. 为b、c注入属性a  -->creatSet中存在a的beanName【判定出现了循环依赖】
         2. 去二级缓存拿a的单例的残缺的bean   --> 此时a尚未初始化，即二级缓存没有-->找三级缓存
         3. 执行三级缓存的lamda表达式 ` getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean)`，判断是否有代理需要(AOP提前执行)生成普通或者代理对象(仍然是**残缺的**)，并将该对象存入earlyProxyReferences这个map
         4. 将此残缺的bean a存入二级缓存，并清空三级缓存保证a的单例性
         5. <u>将残a 赋值给b、c，此时b、c是完整的，进而就可以将b、c赋值给a，</u>**【完工】**
   3. 初始化系列操作
   4. 初始化后(AOP以及其他代理)，由于a已提前完成了AOP，此时不需要了。具体是根据earlyProxyReferences这个map来判断的
   5. 存入单例池



### Bean创建流程

类  -(选择构造)-> 类实例 -> 依赖注入 -> 初始化前(@PostConstruct后置处理)->  初始化(InitializingBean接口) -> 初始化后(AOP) -> 代理对象 ->  存入单例池 -> Bean

+ 选择构造：默认选择无参构造，当没有无参构造时走有参构造，当有参构造也不唯一时，就报错
+ 可通过@Autowired指定要执行的构造方法

+ 当类实例中存在代理对象的属性时，最终就会将代理对象存入单例池
+ 初始化后，如果走了AOP，那么getBean()拿到的就是代理对象。代理对象中所有的属性对象刚开始时是都是空的，即无法调用目标类中依赖于属性对象的方法，但实际上我们又调得通。原因为目标类是普通对象且完成了依赖注入，也就是说目标对象的属性对象是有值的，而代理类会将目标对象以成员属性的方式注入进来。

### Bean生命周期

![image-20220615185132728](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220615185132728.png)

1. 查找并加载需要被Spring管理的bean，进行Bean的实例化
2. Bean实例化后会为bean注入依赖
3. 判断执行Aware【3个】接口的实现方法，进行属性设置。
4. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
5. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
6. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
7. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
8. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。

### @Async与AOP导致的循环依赖问题

在为切片方法加上 @Async 后，抛了循环依赖的异常。

由于三级缓存的时候提前走了AOP，会为AOP生成代理对象并放到二级缓存中去。当进行到初始化后时，会执行大量的BeanPostProcessor方法【AOP在一般情况下的位置也在此处】。由于AOP提前执行过了，此时它会直接返回 [doNothing]；随后会执行@Async对应的BeanPostProcessor方法，但是该方法并没有判断是否执行过Async的代理逻辑，且由于@Async并非常用功能，spring未提供对@Async进行代理对象的提前构建的支持，所以此时会为@Async再次生成一个代理对象。**【问题出现了】：** 这两个代理对象不是同一个对象，spring在判断发现两个对象不相等之后就直接抛异常了。

**解决办法：**

为子属性对象加上@Lazy即可，@Lazy的功效（例如加在b上）：当a注入b时，发现它是懒加载的，就不从单例池中找了，直接生成一个b的代理对象赋值给a。**【注意】：** 这个过程中b没有走正常的创建流程，没有和a打交道，完美避开了循环依赖的路障。 在使用b时才会从单例池中找b，并调用b的方法。但此时 allDone，连a都创建完了，什么BeanPostProcessor都执行过了，就解决了AOP和@Async冲突导致的循环依赖的问题。

### 构造依赖场景导致的循环依赖

当B成为A的构造入参时，spring的正常处理循环依赖的机制以及不起作用了，因为构造A的路上绕不开B【否则构造方法执行不下去】，创建b的时候又需要a，凉凉了。

**解决方法：** 在入参构造上加@Lazy。

### 非单例场景导致的循环依赖

a单b原：a需要b，创建b需要a，a虽然可以从池中拿，但是b无法完成赋值**[原型]**，还是要创建b，凉了。

a原：肯定凉了，因为无法从池中去拿a，第一步都迈不过。



### 事务执行流程

0. **要求为代理对象**

1. 判定存在@Transactional
2. 由事务管理器新建一个数据库连接，并依赖此链接执行sql。为支持并发和多数据源，将其保存起来 `ThreadLocal<Map<DataSource,Connection>>`
3. 修改提交方式为手动提交，改的是事务管理器新建的那个连接
4. 执行切片方法。该过程也依赖数据库连接，所以要确保执行的连接与事务连接是同一个连接，进而要求jdbcTemplate和TransactionalManager要使用同一个DataSource。
5. 判定非否需要回滚



### 事务隔离

代理对象执行目标类方法时，还是由普通对象【非代理对象】执行的，所以在事务通知时会出现隔离事务失效的情况。解决方法，无非是让一个代理的实例来执行这个隔离方法：

+ **方案1 ** 新建一个类，将隔离事务的方法剪过来（带上隔离事务注解），让它走AOP，生产代理类
+ **方案2 ** 将隔离事务对应的类注入进来【即使是当前类也是可以的】，由于当前类存在第一层事务方法，会走AOP生成代理对象，即@Autowired时会将代理对象注入进来【不会循环依赖】，然后显示地用注入的对象来调用就好了。【推荐】



### Spring如何管理事务的

基于三个组件：由TransactionDefinition设置事务属性，并通过 TransactionStatus 实时获取事务状态，最终PlatformTransactionManager根据状态执行提交或者回滚。

**PlatformTransactionManager -- 事务管理器**：主要用于平台相关事务的管理

```
commit 事务提交；
rollback 事务回滚；
getTransaction 获取事务状态；
```

**TransactionDefinition -- 事务定义**：用来定义事务相关的属性，供事务管理器使用

```
getIsolationLevel：获取隔离级别；
getPropagationBehavior：获取传播行为；
getTimeout：获取超时时间；
isReadOnly：是否只读；
```

**TransactionStatus -- 事务具体运行状态**：描述事务管理过程中每个时间点事务的状态信息。

```
hasSavepoint()：返回这个事务内部是否包含一个保存点，
isCompleted()：返回该事务是否已完成，也就是说，是否已经提交或回滚
```

### aop五种通知类型

+ 前置通知，**@Before(value = “”)** 在方法执行前通知
+ 后置通知，**@AfterReturning(value = “”)** 在方法正常执行完成进行通知，可以访问到方法的返回值的。
+ 环绕通知，**@Around(value = “”)** 可以将要执行的方法（point.proceed()）进行包裹执行，可以在前后添加需要执行的操作
+ 异常通知，**@AfterThrowing(value = “”)** 在方法出现异常时进行通知，可以访问到异常对象，且可以指定在出现特定异常时在执行通知。

+ 执行后通知，**@After(value = “”)** 在目标方法执行后无论是否发生异常，执行通知,不能访问目标方法的执行的结果。

**<u>*区分@After和@AfterReturning区别，After不论方法是否执正常结束都会通知，而AfterReturning出现异常未正常结束则不会通知。*</u>**

### *AOP流程和原理



### *Spring事务失效条件

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

### Spring事务的7种隔离级别

可概括为必须用[2]、可以用[3]、不准用[2] 7种情况。

|   PROPAGATION_REQUIRED    |  有事务，就加入当前事务。没有事务就开启一个新事务。  |
| :-----------------------: | :--------------------------------------------------: |
| PROPAGATION_REQUIRES_NEW  | 开启一个新事务，只用自己的事务。其它事务独立与我无关 |
|   PROPAGATION_SUPPORTS    |   有事务，就加入当前事务。没有事务就**不用事务**。   |
|   PROPAGATION_MANDATORY   |    有事务，就加入当前事务。没有事务就**报异常**。    |
|    PROPAGATION_NESTED     |      如果当前存在事务，则运行在一个嵌套的事务中      |
| PROPAGATION_NOT_SUPPORTED |       总是非事务地执行，并挂起任何存在的事务。       |
|     PROPAGATION_NEVER     |  总是非事务地执行，如果存在一个活动事务，则抛出异常  |

### *AOP原理







