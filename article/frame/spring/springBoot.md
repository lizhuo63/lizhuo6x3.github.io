# 特点

1. 创建独⽴的Spring应⽤程序 
2. 直接嵌⼊Tomcat，Jetty或Undertow（⽆需部署WAR⽂件） 
3. 提供依赖项管理，以简化构建配置 
4. 尽可能⾃动配置Spring和第三⽅库 
5. 提供可⽤于⽣产的功能，例如指标，运⾏状况检查
6. 完全没有代码⽣成，也不需要XML配置 

通过spring创建一个简单的HelloWorld的Web项目，大致有以下几步：

​			创建WEB工程

1. 编写web.xml
2. 引入SpringMVC相关第三方包
3. 配置mvc.xml，装备组件
4. 打包
5. 部署到Tomcat

# 应⽤与配置

## 基本结构

```
#⽗依懒
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.4.5</version>
</parent>
#web启动器
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

#启动类和控制器
@RestController
@EnableAutoConfiguration
public class MyApplication {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

**说明：**

1. 父依赖spring-boot-starter-parent（依赖于spring-boot-dependencies）最核心的作用就是解决了插件和依赖的版本兼容问题。

   + spring-boot-starter-parent： 声明了各个插件的版本 

   + spring-boot-dependencies：声明了各个依赖组件的版本 

2. 启动器spring-boot-starter-web：批量引入了与Web相关的依赖

   + 包括spring-web，spring-mvc依赖组件及其他

   + 嵌入了Tomcat（及spring-boot-starter-tomcat启动器）

3. 启动类（@EnableAutoConfiguration），⽤于开启⾃动装配。装配逻辑是：

   <img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220214221134044.png" alt="image-20220214221134044" style="zoom:80%;" />

   <img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220214221248051.png" alt="image-20220214221248051" style="zoom: 80%;" />

   <img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220214221555951.png" alt="image-20220214221555951" style="zoom: 50%;" />

   1. 扫描 spring-boot-autoconfifigure-2.4.0.jarMETA-INF/spring.factories ⽂件
   2. 读取org.springframework.boot.autoconfifigure.***EnableAutoConfifiguration 所有的⾃动装配类 
   3. 判定装配类是否满⾜装配条件? 【1.是否为web项目 ; 2.该类是否在项目中出现过】
   4. 如果满⾜则装配指定@Bean【加入IOC】

   @SpringBootApplication

   <img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220214221017075.png" alt="image-20220214221017075" style="zoom: 67%;" />

​	案例中引⼊了web\mvc等组件，所以DispatcherServletAutoConfifiguration、 WebMvcAutoConfifiguration 就触发装配动作，往Spring ioc中装配 DispatchServlet、RequestMappingHandlerAdapter等WEB相关组件。

4. 打包：由于springboot使用的是内嵌式的tomcat，所以也要使用特殊的插件`spring-boot-maven-plugin`

**运行时jar包的结构**

![image-20220215102932339](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220215102932339.png)

**⽬录说明：** 

+ ./：Spring Boot类装载器 

+ ./META-INF/：JAR 元信息 

+ ./BOOT-INF/classes/：项⽬⾃身类⽂件 

+ ./BOOT-INF/lib/：项⽬依懒JAR包组件 

+ ./BOOT-INF/lib/classpath.idx： 依赖的JAR清单 

+ ./BOOT-INF/lib/layers.idx： 项⽬⽂件清单，⽤于制作Docker镜像 

## 大致流程

![image-20220215103715971](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220215103715971.png)

## 配置体系

### 优先级

**配置优先级：**程序参数>JVM参数>properties文件>yml文件>yaml文件

1.  程序启动参数
2.  JVM参数 
3.  指定⽂件 
4.  指定环境变量 : spring.profiles.active=test
5.  启动⽬录 
6.  config 
7.  properties
8.  yml，yaml

**配置的位置：**

在springboot默认配置⽂件如下，优先级由⾼⾄低。 

<img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220215133625621.png" alt="image-20220215133625621" style="zoom:67%;" />

1. 启动⽬录:config/properties文件>yml文件>yaml文件
2. 启动⽬录:/properties文件>yml文件>yaml文件
3. classpath:config/properties文件>yml文件>yaml文件

4. classpath:/properties文件>yml文件>yaml文件

### 参数(JVM参数\启动参数)

![image-20220215112658587](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220215112658587.png)

```java
public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(HelloApplication.class);
        logger.info(System.getProperty("server.port"));
        System.setProperty("server.port", "8085");
        args = new String[]{"--server.port=8086"};
        SpringApplication.run(HelloApplication.class, args);
    }
```

> + 程序启动参数，通过main方法中args[]参数获取
> + jvm参数, 可通过System.getProperty 获取

### propertites、yml、yaml⽂件配置

​	spring boot 默认会读取 application.properties 配置⽂件，它会覆盖上面的参数配置

```
#引入随机值
${random.int} 
#10以内的随机值
${random.int(10)} 
#50-100随机址
${random.int(50,100)} 
#UUID 
${random.uuid} 
#引入系统变量
${user.name} 
#指定默认值
${user.name:lz}
```



### 配置切换

<img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220215121842742.png" alt="image-20220215121842742" style="zoom:67%;" />

<img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220215121911431.png" alt="image-20220215121911431" style="zoom:67%;" />

**yml**除了上示方式以外，还有以下特有方式。

<img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220215122004990.png" alt="image-20220215122004990" style="zoom:67%;" />



### 读取配置数据

#### @Value

```java
#获取单个数据，⽀持通过#{}进⾏表达示计算，但不⽀持复杂格式。
@Value("${name}")
private String name;
```

#### 为静态属性赋值

1. 

<img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220215132439637.png" alt="image-20220215132439637" style="zoom: 67%;" />


2.

```java
@Autowired
private String t_id;
private static String id;
    @PostConstruct
    public void init() {
        id = t_id;
    }
```

3. 

```java
@ConfigurationProperties(prefix = "xxx")
public class A {
    public static final String ABC = "abc";
只要把set方法设置为非静态，那么这个配置类的静态属性就能成功注入了。
```



#### Environment

```java
@Autowired  
private Environment env;
Environment：全部对应，可通过getProperty获取任意已配置的数据
```



#### @ConfifigurationProperties

```java
#获取符合条件的所有数据，常用于获取对象数据
 @Component
 @ConfigurationProperties(prefix = "admin")
 public class User {
 
 }
```

#### @PropertySource

​		通过@PropertySource 指定⼀个配 置⽂件，⽀持fifile:绝路径，和classpath:路径。 

```java
@PropertySource(value = {"classpath:/user.properties"}, encoding = "UTF- 8")
```

#### 添加验证@Validated

```java
#使⽤前需要添加validation启动器
<dependency>
	<groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

@Component("userInfo") 
@ConfigurationProperties(prefix = "admin")
@Validated 
public class User {
    // 不能为空
    @NotNull
    public String name;
    @DateTimeFormat(pattern = "yyyy-MM-dd") 
    public Date birthday;
    //列表元素不能为空
    @NotEmpty
    public List<String> labels;
    //必须符合邮件格式
    @Email 
    private String email;
}
```

## 注解大全

### 配置相关

#### @SpringBootApplication

​		标识这是一个 Spring Boot 应用的启动类。是**@Configuration,@EnableAutoConfiguration,@ComponentScan**三个注解的组合。

#### @Configuration

​		声明为配置类，初始化一个IOC容器，会将该类下的所有BeanMethod返回结果装到该容器中。

#### @ComponentScan

​		定义扫描路径，把符合扫描规则的类(标识了@Controller，@Service，@Repository，@Component注解的类)装配到spring容器中

#### @EnableAutoConfiguration

​		通过相关组件的AutoConfiguration注解(如：@MybatisAutoConfiguratio)能根据当前类路径下的包或者类来配置 Spring Bean。**须于@Configuration配合使用**

1. 从配置文件META-INF/spring.factories加载所有可能用到的自动配置类
2. 去重，并将exclude和excludeName属性携带的类排除
3. 过滤，将满足条件（@Conditional）的自动配置类返回

####  ＠Component

​		声明为组件，并将该类装进IOC容器

#### @Bean

​		声明一个bean，并交给spring管理。

#### @ConfigurationProperties("***")

​		自定义的properties文件(配置前缀)映射到实体bean中，为其赋值

#### @EnableConfigurationProperties

​		让使用了 @ConfigurationProperties 注解的类生效，将该类注入到 IOC 容器中，须于@Configuration配合使用

#### @Import 

​		导⼊指定配置，该注解可以导⼊任意类 ⾄IOC容器。

#### @AutoWired

​		按类型完成自动装配的工作。当加上（required=false）时，就算找不到bean也不报错。

* **@Qualifier(“name”)**与@Autowired配合使用指定具体装入的bean

#### @Resource

​	按名称完成自动装配的工作。

## 自定义配置

###  IOC容器

​		IOC是一种通过描述来生成或者获取对象的技术，一个系统可以生成各种对象，它们之间还可能存在依赖的关系。可以通过描述管理Bean，包括发布和获取Bean;通过描述完成Bean之间的依赖关系。BeanFactory.getBean()可通过类型和名称获取bean

#### @Configuration创建容器

声明一个Java配置类，Spring的容器会根据它来**<u>生成 IoC 容器</u>**去容纳Bean;

#### 获取配置类容器

```java
#是为被@Configuration标注过的类
ApplicationContext ctx = new AnnotationConfigApplicationContext(APPConfig.class);
ctx.getBean("");
```

#### 获取启动类IOC容器

```java
#创建一个类实现ApplicationContextAware的setApplicationContext方法

@Component
public class SpringUtils implements ApplicationContextAware {

	private static ApplicationContext applicationContext;

	@Override
	public void setApplicationContext(ApplicationContext ctx) throws BeansException {
		if (applicationContext == null) {
			applicationContext = ctx;
		}
	}

	public static Object getBean(String name) {
		return applicationContext.getBean(name);
	}
    
    public static <T> T getBean(Class<T> clazz) {
		return applicationContext.getBean(clazz);
	}
```

### Bean的装配

​		将需要的Bean对象装配到IOC容器中去。

#### @Bean(name="")

​		表示将initPojo方法返回的POJO装配到IOC容器中；name 定义bean的名称

##### ＠Component

​		将该类装进IOC容器；子注解**@value**为bean的属性注入值，会覆盖@Bean的赋值

#### ＠ComponentScan

​		**自身本没有装配功能，指定IOC容器的扫描(装配)目标**【与@Configuration配合使用】，默认只扫描该类所在的包和其所有子包下的装配注解，不添加或添加后不设置任何属性，则不起作用。相当于<u>装配注解的过滤器</u>。

##### 属性

+ **lazyinit = true** 	延迟依赖注入

+ **basePackages/value**	指定扫描的包路径

+ **includeFilters**  	放行扫描

+ **excludeFilters**  	拦截扫描	

  + Fiilter属性

    + type：用来配置Filter的类型，指明过滤方案

      + ANNOTATION :-按照注解类型

        ```java
        扫描目标包下的@Controller注解标注的类
        @ComponentScan(
         basePackages = "com. st.dk.demo6.beans",
         includeFilters = {@ComponentScan.Filter(type= FilterType.ANNOTATION,value= Controller.class)})
        ```

      + ASSIGNABLE_TYPE:指定扫描某个接口下的所有实现类

        ```java
        扫描Animal接口的实现类
        includeFilters = {@ComponentScan.Filter(type= FilterType.ASSIGNABLE_TYPE,value={Animal.class})}
        ```

      + ASPECTJ:指定扫描AspectJ表达式相匹配的类

        ```java
        扫描Animal接口的继承类
        includeFilters = {@ComponentScan.Filter(type= FilterType.ASPECTJ,pattern = "com. st.dk.demo6.beans.Animal+")}
        ```

      + REGEX:指定扫描符合正则表达式的类

        ```java
        扫描以dog结尾的类
        includeFilters = {@ComponentScan.Filter(type= FilterType.REGEX,pattern = ".*.*dog")}
        ```

      + CUSTOM:指定扫描自定义的实现了org.springframework.core.type.filter.TypeFilter接口的类

        ```java
        public class DkFilter  implements TypeFilter {
        @Override
         	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        	//获取当前正在扫描的类的类名
         		String className = 	metadataReader.getClassMetadata().getClassName();
         	//判断是不是Dog的class
        	 if(className.equals(Dog.class.getName())){
         	//返回true表示让spring加载当前的类.
        		return true;
         		 }
         	 //返回false表示不让spring加载当前类
        	return false;
        	 }
         }
        ===============
         includeFilters = {@ComponentScan.Filter(type= FilterType.CUSTOM,value = DkFilter.class)}
        ```

        

    + value：根据type的不同，这个表达式的配置方式也不同。

    + classes：当type为ANNOTATION或者ASSIGNABLE_TYPE时，将对应的类配置在value属性中也可以配置在calsses属性中

    + pattern：当type是REGEX时，将表达式配置的pattern中。

#### Bean的作用域

**Spring环境**

```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

+ singleton  	单例，只存在一个实例
+ prototype	原型，每次取得新实例

**SpringMVC环境**

```java
@Scope(WebApplicationContext.SCOPE_REQUEST) 
```

+ request 	单次请求
+ session 	HTTP会话
+ application  	Web工程
+ global Session  	在一个全局的HTTPSession中，Bean定义对应一个实例

### 依赖注入

​		将需要的Bean注入到当前类中，以供使用

#### @Autowired自动装配

​		先根据类型，再依据名称【必须能且只能找到一个匹配bean，即不允许null，要么找到了，要么报错】，附加属性`@Autowired(required=true)`可允许获取null值。

 **消除歧义性**

+ **@Primary** 

  ​	在实现类或子类上标注，声明该类优先装配

+ @**Qualifier(“name”)**

  ​	配合@Autowired使用，声明以具体名称匹配装配

#### @Resource

​		默认按Bean的名称进行匹配装载。

#### 构造装配

```java
public class Man( @Autowired @Qualifier( "dog”) Animal animal) { 
private Animal animal = null;
this.animal = animal ;}
```

#### 条件装配@Conditional

使用场景：当bean无法装配时就放弃装配，避免程序异常中断;需要声明无法装配的条件

1. 自定义一个**条件类**：实现Condition接口的matches方法，定义跳过的条件

   ```java
   	//取出环境变量
   		Environment env = context.getEnvironment();
   		//判断属性文件是否存在对应的配置
   		return env.containsProperty("database.driverName")
   				&& env.containsProperty("database.url")
   				&& env.containsProperty("database.userNname")
   				&& env.containsProperty("database.password");
   	}
   ```

2. 使用**@Conditional(DatabaseConditiona.class)**标注目标bean

# 框架整合

## Dao

### Mybatis

**依赖**

```xml
<dependencies>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.0</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

**配置** 

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver  
    url: jdbc:mysql://localhost/***?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
    username: root
    password: 123456
    
mybatis:
  type-aliases-package:
  mapper-locations:
  configuration:
    ****: ***

#mybatis.config-location=classpath:mybatis/mybatis-config.xml
```

**⾃定义配置MyBatis**

* 配置⽂件mybatis.confifiguration.* 可进⾏全⽅位配置。它配置的就是confifiguration类。

* 通过mybatis.confifig-location 指定mybatis-confifig.xml ，按传统⽅式进⾏配置。

* 通过注⼊ ConfifigurationCustomizer Bean 以代码⽅式 设定confifiguration，它可以和第⼀种⽅式融合，并且会覆盖第⼀种配置。 

  ```java
  @Configuration
  public class MyBatisConfig {
      @Bean
      public ConfigurationCustomizer mybatis(){
          return configuration -> {
            configuration.set***();
          };
      }
  }
  ```

  

**注意：**默认情况下代码包下的XML不会被编译到输出⽬录，需要在POM中配置；要么将mapper.xml放到classpath:/resources/mapper/下

```xml
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/test/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```

#### 集成原理

**MyBatis 核⼼组件**

* Confifiguration ：整体配置，作⽤域为整个应⽤ 
* SqlSessionFactory：会话⼯⼚，作⽤域为整个应⽤ 
* SqlSession：Sql会话，作⽤域为单个请求（不能跨线程） 
* Mapper接⼝：⽤户接⼝，，作⽤域等同于会话 

springboot在过mybatis-spring-**.jar之上与mybatis集成。

![image-20220218165131873](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220218165131873.png)

![image-20220218170802414](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220218170802414.png)

**详细过程**

1. 初始化：使⽤ SqlSessionFactoryBean 创建SqlSessionFactory 以及Confifiguration 

2. 扫描Mapper:通过MapperScannerConfifigurer 扫码指定包下的Mapper 类，然后利⽤Spring 后置处理器动态注⼊MapperFactoryBean 定义 
   1. MapperFactoryBean创建Mapper实例，最终实现还是⽤的Confifiguration 创建，指定会话 为SqlSessionTemplate 
   2. ⽤户基于IOC引⽤Mapper实例 

3. Sql调⽤ 
   1. 执⾏Mapper⽅法 
   2. SqlSessionTemplate 处理请求，基于代理拦截调⽤ 
   3. 重新打开实际会话 

因为每交执⾏⽅法 都是重新开启MyBatis会话，所以Spring创建的Mapper实例是可以跨线程的。

### Redis

**依赖**

```xml
 <!--redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
<!--连接池-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>

```

**参数配置**

```yaml
 redis:
    host: 172.18.0.2
    port: 6379
    timeout: 10000ms  #连接超时时间
    lettuce:
      pool:
        max-active: 1024 #最大连接数 默认8
        max-wait: 1000ms #最大连接阻塞等待时间
        min-idle: 5 #最x空闲连接数
        max-idle: 200 #最大空闲连接  
```

**配置类**

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        //String-key序列器
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        //String-value序列器
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        //Hash-key序列器
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        //Hash-value序列器
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        return redisTemplate;
    }
}
```

[**常用RedisTemplate_Api**](https://docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/core/RedisTemplate.html)

## View

###FreeMarker

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>
```

## task

**cron表达式**

![image-20220316114607810](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220316114607810.png)

### Schedul

```java
@SpringBootApplication
@EnableScheduling//开启定时任务
public class DemoTaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoTaskApplication.class, args);
    }

    @Scheduled(cron = "0/3 * * * * ?")//设置触发机制
    public void t1() {
        System.out.println("t1"+"    "+ LocalTime.now().getSecond());
    }

    @Scheduled(cron = "0/5 * * * * ?")
    public void t2() {
        System.out.println("t2"+"    "+LocalTime.now().getSecond());
    }

}
```

### quartz

#### 核心概念

+ **job：**任务类，需要实现job接口的excute()方法

+ **Trigger：**触发器
+ **Scheduler：**调度器，整合了job和Trigger，在指定需求下触发执行任务

![image-20220316115923871](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220316115923871.png)

**Quartz的重要组件：** 

+ Scheduler：用于与调度程序交互的主程序接口。 
  Scheduler调度程序-任务执行计划表，只有存入执行计划表的任务job(通过scheduler.schedulelob方法存入执行计划)，才会触发trigger。
+ job：能被调度程序执行的任务类，可自定义。 
+ JobDetail：使用jobDetail来定义定时任务的实例，JobDetail实例是通过obBuilder类创建的。 
+ JobDataMap：可以包含不限量的（序列化的）数据对象，在job实例执行的时候，可以使用其中的数据。 
+ Trigger：触发器，用来触发job的执行。当调度一个job时，需要实例一个触发器然后调整它的属
  性来满足job执行的条件。表明任务在什么时候会执行。
+ JobBuilder：用于声明一个任务实例，也可以定义关于该任务的详情比如任务名、组名等。 
+ TriggerBuilder：触发器创建器，用于创建触发器trigger实例。 
+ JobListener、TriggerListener、SchedulerListener：监听器，用于对组件的监听。





## Docker

1. 在宿主机上创建⼀个应⽤⽬录，并拷⻉打构建好的可执⾏jar⾄当前⽬录

```
mdir -p boot-demo/config 
mdir -p boot-demo/logs
```

2. 创建dockerfifile 

```
FROM java:8
ADD boot-demo.jar /app.jar
ENTRYPOINT ["java" ,"-jar", "/app.jar"]
```

3. 启动容器 

   ```
   docker run --name boot-demo -v /home/lz/public/boot-demo/config:/config -v /home/lz/public/boot-demo/logs/:/logs -p 8081:8081 boot-demo
   ```

4. 启动&停⽌ 

   ```
   docker stop boot-demo
   docker start boot-demo
   ```

   

# 框架原理

## 启动器

启动器是指对某个场景的启动，内容包括： 

1. 依懒环境引⼊ 

2. 配置信息引⼊ 

3. ⾃动装配Bean 

![image-20220218204718477](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220218204718477.png)

### 实现过程

1. 添加环境依赖

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.4.0</version>
</dependency>
```

2. 配置属性⽂件 

```JAVA
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@ConfigurationProperties("XXX")
public class ApiProperties {
    private String context="/XXX";

    public String getContext() {
        return context;
    }

    public void setContext(String context) {
        this.context = context;
    }
}
```

3.  设置装配类 

```JAVA
@Configuration
@EnableConfigurationProperties(DemoProperties.class) // 自动注入配置Bean
@ConditionalOnWebApplication
public class DemoAutoConfiguration {
    //1.配置文件
    ApiProperties properties;
    static Logger logger = LoggerFactory.getLogger(DemoAutoConfiguration.class);

    public DemoAutoConfiguration(DemoProperties properties) {
        this.properties = properties;
    }

```

4. 添加引导⽂件 

   resources/META-INF/spring.factories 

   ```
   # Auto Configure
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   lz.demo.DemoAutoConfiguration
   ```

## 可执⾏JAR包

​		./BOOT-INF/classes/:(项⽬⾃身类⽂件)下的启动类由于找不到./BOOT-INF/lib/:(项⽬依懒JAR包组件)下的class文件，所以需要借助外力。

![image-20220218231839680](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220218231839680.png)

## spring轻配置模拟

