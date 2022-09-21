## Bean作用域

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_22-18-44.png)

##  Bean生命周期

1. 实例化Bean
2. 依赖注入
3. 检查该Bean实现的相关Aware接口，并设置对应的资源
4. 检查该Bean是否自定义 init-method，有就调用
5. 检查该Bean是否实现了BeanPostProcessor 接口，将会调用postProcessAfterIn
   itialization(Object obj, String s)方法
6. Bean初始化成功，投入使用
7. 检查该Bean是否实现了DisposableBean这个接口，有就调用destroy()方法
8. 最后检查这个Bean是否自定义了destroy-method属性，有就调用自配置的销毁方法

## IOC

IOC 即“控制反转”，是一种设计思想，它利用java 反射机制。以前要由程序主动去创建依赖对象，耦合性很强，IOC 则是由容器来帮忙创建及注入依赖对象，对象只是被动的接受依赖对象。

DI即“依赖注入”**：**组件之间的依赖关系由容器在运行期决定的，即由容器动态的将依赖关系注入到组件之中。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

## AOP

意为面向切面编程，是OOP的延续，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。可用于日志记录、效率检查、事务控制。

## 循环依赖

- ①构造器的循环依赖：这种依赖spring是处理不了的，直接抛出BeanCurrentlylnCreationException异常。
- ②单例模式下的setter循环依赖：通过“三级缓存”处理循环依赖，能处理。
- ③非单例循环依赖：无法处理。原型(Prototype)的场景是不支持循环依赖的，通常会走到`AbstractBeanFactory类中下面的判断，抛出异常。`

**三级缓存：**

+ singletonObjects （一级缓存）俗称“单例池”“容器”，用于缓存创建完成的单例Bean
+ earlySingletonObjects（二级缓存）映射Bean的早期引用(不是完整的)，只是一个Instance.
+ singletonFactories（三级缓存） 映射创建Bean的原始工厂

当 Spring 为某个 Bean 填充属性时，首先会寻找需要注入对象的名称，然后依次执行 getSingleton() 方法得到所需注入的对象，而获取对象的过程就是依次从一级缓存、二级缓存中获、三级缓存中获取，如果三级缓存中也没有，那么就会去执行 doCreateBean() 方法创建这个 Bean。


# spring

### Bean

#### 创建流程

类  -(选择构造)-> 类实例 -> 依赖注入 -> 初始化前(@PostConstruct后置处理)->  初始化(InitializingBean接口) -> 初始化后(AOP) -> 代理对象 ->  存入单例池 -> Bean

+ 选择构造：默认选择无参构造，当没有无参构造时走有参构造，当有参构造也不唯一时，就报错
+ 可通过@Autowired指定要执行的构造方法

+ 当类实例中存在代理对象的属性时，最终就会将代理对象存入单例池
+ 初始化后，如果走了AOP，那么getBean()拿到的就是代理对象。代理对象中所有的属性对象刚开始时是都是空的，即无法调用目标类中依赖于属性对象的方法，但实际上我们又调得通。原因是目标类是普通对象且完成了依赖注入，也就是说目标对象的属性对象是有值的，代理类会将目标对象以成员属性的方式注入进来。

#### 循环依赖

##### Spring方案

示例：A-->B,C；B-->A；C-->A，设先创建A

单例池（一级缓存）**SingletionObjects** ：存储完整的单例bean

二级缓存 **earlySingletionObjects**：保存没有经过完整生命周期的bean，属性值为null，同时保证该bean即使不完整也是单例的。它主要用于保证B、C能够拿到同一个A。

三级缓存 **SingletionFactories**：在bean创建之初就不管是否有循环依赖，都会根据beanName，beanDefinition和当前的普通bean生成一个lamda表达式，存入三级缓存。**它是打破循环的关键。** 该表达式也是AOP的执行入口，核心作用是动态判断生成一个普通对象或是代理对象。

1. 创建A

    0. alreadyCreated("a")，记录下来

    1. 创建a普通对象 --> [此时A中的b、c均为null]根据当前A的残缺的bean、beanName、beanDefinition生成一个lambda表达式
    2. 注入属性，B、C
        1. 去单例池中找b、c ，没有就创建b、c
            1. 为b、c注入属性a  -->creatSet中存在a的beanName【判定出现了循环依赖】
            2. 去二级缓存拿a的单例的残缺的bean   --> 此时a尚未初始化，即二级缓存没有-->找三级缓存
            3. 执行三级缓存的lamda表达式 ` getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean)`，判断是否有代理需要(AOP提前执行)生成普通或者代理对象，**残缺的**，并将该对象存入earlyProxyReferences这个map
            4. 将此残缺的bean a存入二级缓存，并清空三级缓存保证a的单例性
            5. <u>将残a 赋值给b、c，此时b、c是完整的，进而就可以将b、c赋值给a，</u>**【完工】**
    3. 初始化系列操作
    4. 初始化后(AOP)，由于a已提前完成了AOP，此时不需要了。具体是根据earlyProxyReferences这个map来判断的
    5. 存入单例池

##### @Async与AOP导致的循环依赖问题

在为切片方法加上 @Async 后，抛了循环依赖的异常。

由于三级缓存的时候提前走了AOP，会为AOP生成代理对象并放到二级缓存中去。当进行到初始化后时，会执行大量的BeanPostProcessor方法【AOP在一般情况下的位置也在此处】。由于AOP提前执行过了，此时它会直接返回 [doNothing]；随后会执行@Async对应的BeanPostProcessor方法，但是该方法并没有判断是否执行过Async的代理逻辑，且由于@Async并非常用功能，spring未提供对@Async进行代理对象的提前构建的支持，所以此时会为@Async再次生成一个代理对象。**【问题出现了】：** 这两个代理对象不是同一个对象，spring在判断发现两个对象不相等之后就直接抛异常了。

**解决办法：**

为子属性对象加上@Lazy即可，@Lazy的功效（例如加在b上）：当a注入b时，发现它是懒加载的，就不从单例池中找了，直接生成一个b的代理对象赋值给a。**【注意】：** 这个过程中b没有走正常的创建流程，没有和a打交道，完美避开了循环依赖的路障。 在使用b时才会从单例池中找b，并调用b的方法。但此时 allDone，连a都创建完了，什么BeanPostProcessor都执行过了，就解决了AOP和@Async冲突导致的循环依赖的问题。

##### 构造依赖场景

当B成为A的构造入参时，spring的正常处理循环依赖的机制以及不起作用了，因为构造A的路上绕不开B【否则构造方法执行不下去】，创建b的时候又需要a，凉凉了。

**解决方法：** 在入参构造上加@Lazy。

##### 非单例场景导致的循环依赖

a单b原：a需要b，创建b需要a，a虽然可以从池中拿，但是b无法完成赋值**[原型]**，还是要创建b，凉了。

a原：肯定凉了，因为无法从池中去拿a，第一步都迈不过。

### 事务

#### 执行流程

0. **要求为代理对象**

1. 判定存在@Transactional
2. 由事务管理器新建一个数据库连接，并依赖此链接执行sql。为支持并发和多数据源，将其保存起来 `ThreadLocal<Map<DataSource,Connection>>`
3. 修改提交方式为手动提交，改的是事务管理器新建的那个连接
4. 执行切片方法。该过程也依赖数据库连接，所以要确保执行的连接与事务连接是同一个连接，进而要求jdbcTemplate和TransactionalManager要使用同一个DataSource。
5. 判定非否需要回滚



#### 事务隔离

代理对象执行目标类方法时，还是由普通对象【非代理对象】执行的，所以在事务通知时会出现隔离事务失效的情况。解决方法，无非是让一个代理的实例来执行这个隔离方法：

+ **方案1 ** 新建一个类，将隔离事务的方法剪过来（带上隔离事务注解），让它走AOP，生产代理类
+ **方案2 ** 将隔离事务对应的类注入进来【即使是当前类也是可以的】，由于当前类存在第一层事务方法，会走AOP生成代理对象，即@Autowired时会将代理对象注入进来【不会循环依赖】，然后显示地用注入的对象来调用就好了。【推荐】

