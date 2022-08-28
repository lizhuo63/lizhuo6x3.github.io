# Bean

## 定义Bean的方式

### 1. 基于XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd">
<!-- 定义一个 Bean 的信息 -->
<bean id="car" class="com.demo.compent.Car"></bean>
</beans>
```

### 2. 基于配置类，需注解配合

#### @Configuration

#### @Bean

#### @Component

```java
@Configuration
@ComponentScan(basePackages = {"com.demo.testcompentscan"})
public class MainConfig {
		@Bean
		public Person person(){
				return new Person();
		}
}
```

### 3. 直接调用IOC.registerBean()

【需要对IOC容器进行刷新】

```java
	public static void main(String[] args) {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
		context.registerBean(Hello.class);
*		context.refresh();
		Hello hello = (Hello) context.getBean("hello");
		System.out.println(hello);
	}
```

### 4. 创建BeanDefinition

需要借助BeanDefinitionBuilder获取beanDefinition对象，再将该对象设置到IOC容器中。【需要对IOC容器进行刷新】

```java
public static void main(String[] args) {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    // 创建一个beanDefinition，并设置它对应的bean的类型
		AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition().getBeanDefinition();
		beanDefinition.setBeanClass(MoBian.class);
    // 将beanDefinition注册到Spring容器中，并设置对应的Bean的名字
		context.registerBeanDefinition("mobian",beanDefinition);
*		context.refresh();
		System.out.println(context.getBean("mobian"));
	}
```



## Bean的核心属性

### 1. @Scope 作用域

+ **singleton** 单实例的**(默认)** ：饿汉式加载（容器启动后Bean实例就全部创建好了）
+ **prototype** 多实例的 ： 懒汉式加载（在第一次使用的时候才会去创建）
+ request 同一次请求
+ session 同一个会话级别

### 2. @Lazy

关键功能是该改变单例Bean的创建实际，让其在在第一次使用的时候才会去创建



## Bean的注入方式

```java
public class User {
    private String name;
    private int age;
    private Car car;
}
```

### 1. set注入

```xml
<bean id="user" class="cn.tewuyiang.pojo.User">
    <property name="name" value="aaa" />
    <property name="age" value="123" />
*   <property name="car" ref="myCar" />
</bean>
```

###  2. 构造器注入

```xml
<bean id="user" class="cn.tewuyiang.pojo.User">
    <constructor-arg name="name" value="aaa" />
    <constructor-arg name="age" value="123" />
    <constructor-arg name="car" ref="myCar" />
</bean>
```

### 3. 静态工厂

```java
public class SimpleFactory {
		//静态工厂，返回一个Car的实例对象
    public static Car getCar() {
        return new Car(12345, 5.4321);
    }
}

<bean id="car" class="cn.tewuyiang.factory.SimpleFactory" factory-method="getCar"/>
```

### 4. 实例工厂

实例工厂与静态工厂不同的是，实例工厂需要有一个实例对象才能调用它的工厂方法。

```java
public class SimpleFactory {
    //实例工厂方法，返回一个Car的实例对象
    public Car getCar() {
        return new Car(12345, 5.4321);
    }
    //实例工厂方法，返回一个String
    public String getName() {
        return "tewuyiang";
    }
     //实例工厂方法，返回一个int，在Spring容器中会被包装成Integer
    public int getAge() {
        return 128;
    }
}
```

### 5. 注解注入

使用@Autowired 、@Resource注入引用类型，通过@Value注入基本类型和数组以及集合。



## 创建Bean实例的方法



## Bean的声明周期流程

1. 扫描项目，将配置类中指定路径范围下的Class文件转换为Class对象
2. 将Class对象属性包装为`BeanDefinition`，然后存入 BeanDefinitionMap 中！
3. 在所有的`BeanDefinition`生成和保存执行完毕后，开始调用第一个回调接口`BeanFactoryPostProcessor#postProcessBeanFactory()`! **它是在将Class文件转换为 `BeanDefinition` 之后调用的**，通过回调此方法可以获取所有的`BeanDefinition` ，可以通过修改它，来改变后续的流程！
4. 解析 `BeanDefinition` 对象中的Bean信息
5. 





## 判断注册Bean的方式

### 1. @CompentScan

+ 如果不设置value属性，默认扫描路径是启动类 XxxApplication.java 所在目录及其子目录

支持按照 **ANNOTATION【注解类型】、ASSIGNABLE_TYPE【指定类型】、**ASPECTJ【按照AOP切入表达式】、REGEX【按照正则表达式】、CUSTOM【自定义规则】等五种方式来过滤。

```java
@Configuration
@ComponentScan(basePackages = {"com.demo.testcompentscan"},excludeFilters = {
							@ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Controller.class}),
							@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,value = {TestService.class})
							})
public class MainConfig {
}
```

### 2. @Conditional 条件判断

可作用在类和方法上，用于动态判断是否将目标Bean注册到IOC中

**实现步骤：**

1. 定义判断条件，需要实现 Condition 接口

```java
public class TestCondition implements Condition {
		@Override
		public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
				//判断容器中是否有 TestAspect 的组件
				if(context.getBeanFactory().containsBean("testAspect")) {
							return true;
				}else return false;
		}
}

```

2. 使用

```java
public class MainConfig {
		@Bean
		public TestAspect testAspect() {
				return new TestAspect();
		}
		//当切 容器中有 testAspect 的组件，那么 testLog 才会被实例化.
		@Bean
		@Conditional(value = TestCondition.class)
		public TestLog testLog() {
				return new TestLog();
		}
}
```





## 将组件注册到IOC容器方式

### 移交 类【实例化流程由Spring管制】

#### 1. @CompentScan +@compent

适用于第二方的Java类，即源码可更改的、用户自己手写的创建

#### 2. @Import

适用于第二方的Java类

```java
@Configuration
@Import(value = {Person.class, Car.class}) //全类名路径
public class MainConfig {
}
```

### 移交 实例【实例化流程可自主控制】

#### 3. @Bean、XML （支持传参，但无法批量）

适用于第三方的依赖，常常是 先new后@Bean。

#### 4. registerSingleton( );

它是直接将一个实例对象交给spring来管理，只要该实例 **[可 new()，可代理，可反射]** 生成成功，就可以100%注册到IOC中。而上述两种是通过扫描类，将类交给Spring来管理，该类的实例化过程将受到spring的管制，也就是说，是不是要实例化该类并把它注册到IOC中去，完全由Spring来决定。

```java
public static void main(String[] args) {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
		context.getBeanFactory().registerSingleton("men",new Men());
    context.refresh();
	}
```

#### 5. 实现FactoryBean

Spring通过反射机制利用<bean>的class属性来实例化Bean，然而bean具有大量的配置属性，实例化Bean过程也就比较复杂，为简化这一过程，Spring提供了一个FactoryBean的工厂类接口，专门用于获取指定类型的Bean实例。

```java
public class CarFactoryBean implements FactoryBean<Car> {
		//返回 bean 的对象
		@Override
		public Car getObject() throws Exception {
				return new Car();
		}
		//返回 bean 的类型
		@Override
		public Class<?> getObjectType() {
				return Car.class;
		}
		//是否为单例
		@Override
		public boolean isSingleton() {
				return true;
		}
}
```

#### 6. 实现 ImportBeanDefinitionRegistrar

可对目标类构建BeanDefinition，支持参数设置，功能强大，但是它调用的是目标类的无参构造。需@Import导入配置类

```java
public class MockImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
	//调用是默认的无参构造,但又需要设置参数，只能是 propertyValues.add(k,v); 【注意：若mockFactoryBean没有无参构造就报错】
	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		//模拟扫描后的批量
		ArrayList<Class> classes = new ArrayList<Class>();
		classes.add(MenMapper.class);
		classes.add(UserMapper.class);
		//注册所有
		for (Class clazz : classes) {
			BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(MockFactoryBean.class);
			AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
			MutablePropertyValues propertyValues = beanDefinition.getPropertyValues();
			propertyValues.add("mapperInterface", clazz);
			//别名不能重复
			registry.registerBeanDefinition(clazz.getSimpleName(), beanDefinition);
		}
	}
}
```



## 自定义Bean的生命周期函数

**1.2.3均是创建后和销毁前，而4是创建前和创建后。**

### 1. 设置@Bean的属性值

1. 声明初始化和销毁方法

```java
@Configuration
public class MainConfig {
		//指定了 bean 的生命周期的初始化方法和销毁方法.分别在car创建后和容器销毁前调用
		@Bean(initMethod = "init",destroyMethod = "destroy")
		public Car car() {
				return new Car();
		}
}
```

2. 在Car类中创建对应的方法

### 2. 使用@PostConstruct、@ProDestory 注解

```java
@Component
public class Book {
		public Book() {
				System.out.println("book 的构造方法");
		}
		@PostConstruct
		public void init() {
    		System.out.println("book 的 PostConstruct 标志的方法");
		}
		@PreDestroy
		public void destory() {
				System.out.println("book 的 PreDestory 标注的方法");
		}
}
```

### 3. 实现InitializingBean、DisposableBean接口

```java
@Component
public class Person implements InitializingBean,DisposableBean {
		public Person() {
				System.out.println("Person 的构造方法");
		}
		@Override
		public void destroy() throws Exception {
				System.out.println("DisposableBean 的 destroy()方法 ");
		}
		@Override
		public void afterPropertiesSet() throws Exception {
    		System.out.println("InitializingBean 的 afterPropertiesSet 方法");
		}
}
```

### 4. 实现BeanPostProcessor

**【注意】：**以下两个方法分别在 **init** 方法之前和之后调用，而且会拦截所有的Bean

```java
public class TestBeanPostProcessor implements BeanPostProcessor {
@Override
		public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("TestBeanPostProcessor...postProcessBeforeInitialization:"+beanName);
				return bean;
		}
		@Override
		public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
			System.out.println("TestBeanPostProcessor...postProcessAfterInitialization:"+beanName);
					return bean;
		}
}
```



##  bean的实例化流程

1. IOC容器启动后，默认会加载5个自用的Class。
2. 根据附带(配置或注解)的元数据，对指定的包或Class进行扫描，解析被扫描到的Class，将它们的信息存储到 BeanDefinition 中，【它是Bean的原材料，存储有bean的各种属性】
3. 将 BeanDefinition 存入BeanDefinitionMap中，并检测是否有自定义的 BeanFactoryPostProcessor 存在，如果有，就对BeanDefinitionMap 中的BeanDefinition 进行后置处理。【BeanDefinitionMap是一个ConcurrentHashMap，存在于DefaultListableBeanFactory中】
4. 注册所有的 BeanPostProcessor 到容器内部
5. 初始化国际化、事件 资源。
6. 实例化Class
7. 自动注入。
8. 回调BeanPostProcessors.postProcessBeforeInitialization方法
9. 调用bean的初始化方法
10. 回调BeanPostProcessors.postProcessAfterInitialization方法



# IOC

在spring中，所有的资源对象都用Bean来描述，IOC就是Bean的容器，它的思想核心在于：资源不由使用资源的双方管理，而由不使用资源的第三方管理，这么做的好处是 **1.** 资源集中管理   **2.** 降低了双方对资源使用的耦合度。

## IOC体系的核心类

### BeanFactory【容器族】

BeanFactory会将 Bean定义配置成Bean，并提供对Bean的访问。



### BeanFactoryPostProcessor【配置族】

![image-20220820133901209](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220820133901209.png)

允许**修改IOC中的Bean定义**，进而更新 Bean工厂中Bean的属性值。但不能与Bean 实例交互。这样做可能会导致过早的 bean 实例化，违反容器并导致意外的副作用

#### BeanDefinitionRegistryPostProcessor

在 BeanFactoryPostProcessor 方法执行之前，能对配置类扫描生成的BeanDefinition进行更新并注册到DefaultListableBeanFactory中的BeanDefinitionMap里。

#### ConfigurationClassPostProcessor

配置类后置处理器，有强大的配置类注解解析功能，**【在Spring中十分重要】**：

+ 解析@ComponentScan注解，扫描@Configuration、@Component注解并注册BeanDefinition
+ 解析@Import注解，然后进行实例化，并执行ImportBeanDefinitionRegistrar的registerBeanDefinitions逻辑，或者ImportSelector的selectImports逻辑
+ 解析方法级别@Bean注解，并将返回值注册成BeanDefinition
+ ......

##### 扫描原理：

ConfigurationClassPostProcessor先去扫描@Configuration，然后处理它@PropertySource，@ComponentScan，@Import，@ImportResource，@Bean。递归下去直到扫描完毕。这些类被处理后都放入spring容器当中。

@PropertySource：读取配置文件
@ComponentScan：扫描相应包下的注解类，或者扫描指定的类
@ImportResource：导入资源文件
@Bean：实例化一个bean放入spring容器当中。
@Import：实例化一个指定的类，当这个类没有实现ImportSelector或者ImportBeanDefinitionRegistrar，会把它当中一个普通的@Configuration来处理。


#### ImportBeanDefinitionRegistrar

与以上三者绝然不同，它支持对 BeanDefinition 的新值和移除，支持扩展，强大！,

## IOC初始化流程 {5.3.22}

```java
@ComponentScan("spring")
public class APP {
	public static void main(String[] args) {
    // 创建IOC实现类的实例
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(APP.class);
		User bean = context.getBean(User.class);
	}
}
```

**流程梗概：**

```java
public AnnotationConfigApplicationContext(Class<?>...componentClasses) {
 	  this(); // 执行默认的构造【存在父类会先执行父类的默认构造】
    this.register(componentClasses); // 注册配置类，完善IOC环境
    this.refresh(); // 刷新IOC容器，扩展Bean、更改Bean定义
    }
```

### [1] this(); 初始化IOC容器

```java
public AnnotationConfigApplicationContext() {
   StartupStep createAnnotatedBeanDefReader = this.getApplicationStartup().start("spring.context.annotated-bean-reader.create");
   //为IOC容器实例化 注解的Bean定义读取器，此过程中会将Spring指定的5个BeanDefinition注册到BeanDefinitionMap中
   this.reader = new AnnotatedBeanDefinitionReader(this);
   createAnnotatedBeanDefReader.end();
   //为IOC容器实例化 类路径下的bean定义的扫描器
   this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

#### [1.1] 调用GenericApplicationContext的默认构造

该过程会实例化诸多集合

![image-20220820115036829](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220820115036829.png)

该过程结束后，IOC容器就创建好了。

#### [1.2] 调用 new AnnotatedBeanDefinitionReader(this);

为IOC容器实例化 注解的Bean定义读取器，并**在BeanDefinitionMap中注册5个默认的 RootBeanDefinition。**

![image-20220820131302620](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220820131302620.png)

![image-20220812234847703](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220812234847703.png)

这5个BeanDefinition是用户在使用Spring时都会使用到的，它们分别对应的Class是：

+ **ConfigurationClassPostProcessor**：配置类后置处理器，支持对BeanDefinition的维护，常见功能
  + 解析@ComponentScan注解，扫描@Configuration、@Component注解并注册BeanDefinition
  + 解析@Import注解，然后进行实例化，并执行ImportBeanDefinitionRegistrar的registerBeanDefinitions逻辑，或者ImportSelector的selectImports逻辑
  + 解析方法级别@Bean注解，并将返回值注册成BeanDefinition
  + ......（是不是很重要啊！）
+ **AutowiredAnnotationBeanPostProcessor：** AutoWired注解解析器（的bean定义后置处理器）
  +  支持 `@Autowired` 和 `@Value` 注解的支持
+ **CommonAnnotationBeanPostProcessor：**
  + 支持对`@PostConstruct`和`@PreDestroy`，@Resource注解的处理
+ 



### [2] register(componentClasses); 注册配置类

#### [2.1] new AnnotatedGenericBeanDefinition(beanClass);

根据配置类Class生成对应的BeanDefinition，并在后续的一系列方法中为该BeanDefinition设置属性。

#### [2.2] BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);

将配置类对应的BeanDefinition注册到BeanDefinitionMap中。

### [3] refresh();刷新容器

会去执行AbstractApplicationContext#refresh( )方法，这是一个同步方法。

#### [3.1] 前置的准备

在将目标Class 生成BeanDefinition并注册到BeanDefinitionMap之前，会执行一些前置方法，其中

**prepareBeanFactory(beanFactory);**中核心代码如下：

![image-20220820230529262](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220820230529262.png)

会将以下4个与环境相关的Class注册到SingletonObjects中：

![image-20220820230736021](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220820230736021.png)

#### [3.2] invokeBeanFactoryPostProcessors(beanFactory);

根据配置类、及自定义扩展执行扫描，找出所有beanFactory后置处理器，并且调用它们的实现来改变Bean定义，最后注册到BeanDefinitionMap中。此处采用的是分批分步执行的策略：

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-08-21_14-03-42.png)

+ **【注意】: 除去自定义扩展的，都会在记录之前直接执行getBean()提前实例化**

```java
	public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
		// 第一批 首先处理 BeanDefinitionRegistryPostProcessors
    //用于存放所有扫描到的BeanDefinition
		Set<String> processedBeans = new HashSet<>();
		//由于此时仍然是处理BeanDefinition，故一定会步入此判断
		if (beanFactory instanceof BeanDefinitionRegistry) {
      //作此转型，便于处理BeanDefinition
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
      //存储所有注册过的 BeanFactoryPostProcessorp【直接实现的，不含BeanDefinitionRegistryPostProcessor】
			List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
      //存储所有注册过的 BeanDefinitionRegistryPostProcessor
			List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();
			//由于此时还未执行扫描，自定义的beanFactoryPostProcessor是没有的，所以是对Spring内置的beanFactoryPostProcessors的遍历【但是当前还是BeanDefinition状态，所以是没有的】，另外就是对外接第三方的拓展【例如：context.addBeanFactoryPostProcessor(new MaybatisBeanFactoryPostProcessor());】而在此处我没有任何扩展，所以一般会跳过
			for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
          //直接调用实现类的方法，率先执行第三方扩展(非扫描)的 BeanDefinitionRegistryPostProcessor的postProcessBeanDefinitionRegistry()方法，注意只是执行方法，并未注册到Map
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
          //将它记录下来
					registryProcessors.add(registryProcessor);
				}
				else {
          //对于第三方扩展的 BeanFactoryPostProcessor，只是记录，暂时不执行
					regularPostProcessors.add(postProcessor);
				}
			}
			//用于暂存当前需要注册的 BeanDefinitionRegistryPostProcessor，因为要分三步进行注册
			List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();
			String[] postProcessorNames =
					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
        //先注册实现过PriorityOrdered的 BeanDefinitionRegistryPostProcessor，【注意：由于当前还未执行扫描，所以@Component添加的还不在其列，唯有ConfigurationClassPostProcessor独一份】
				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
          //先执行 ConfigurationClassPostProcessor 的实例化并存入singletonObjects，然后再记录
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
      //真正执行前，将当前所有的任务清单单独保存下来
			registryProcessors.addAll(currentRegistryProcessors);
      //执行扫描，加载配置，并将扫描后生成的所有BD注册到BDMap中【此乃注解配置的必经之路。期间会解析@Import】
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
      //清空当前任务，为下一轮做准备
			currentRegistryProcessors.clear();
			//第二轮
			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
        //执行没有在上一轮被执行过，但实现了Ordered的 BeanDefinitionRegistryPostProcessor
				if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
				  //先执行实例化存入singletonObjects，然后再记录
          currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
      //会先执行实现了PriorityOrdered的，再执行实现Ordered的 BeanDefinitionRegistryPostProcessor
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
			currentRegistryProcessors.clear();

      //第三轮
			boolean reiterate = true;
			while (reiterate) {
				reiterate = false;
				postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
				for (String ppName : postProcessorNames) {
          //执行没有在上两轮被执行过的剩余的所有的 BeanDefinitionRegistryPostProcessor
					if (!processedBeans.contains(ppName)) {
            //先执行实例化存入singletonObjects，然后再记录
						currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
						processedBeans.add(ppName);
						reiterate = true;
					}
				}
				sortPostProcessors(currentRegistryProcessors, beanFactory);
				registryProcessors.addAll(currentRegistryProcessors);
			  invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
				currentRegistryProcessors.clear();
			}
			//执行所有【手动扩展和注解扫描的】BeanDefinitionRegistryPostProcessor的postProcessBeanFactory()方法,【先执行扩展的，后执行扫描的】
			invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
      //执行所有手动扩展的 BeanFactoryPostProcessors 的 postProcessBeanFactory 方法
			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
		}

		else {
			invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
		}
    // 第二批 处理 BeanFactoryPostProcessor
		//去IOC容器中获取直接实现 BeanFactoryPostProcessor 类型的，【都是扫描得到的，手动扩展的已经执行完了】
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);
    //分离实现了 PriorityOrdered接口的、 Ordered接口的 以及普通的
		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (processedBeans.contains(ppName)) {
				// skip - already processed in first phase above
			}
			else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}
    //执行实现了 PriorityOrdered接口的 BeanFactoryPostProcessor的 postProcessBeanFactory方法
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);
    
		//执行实现了 Ordered接口的 BeanFactoryPostProcessor的 postProcessBeanFactory方法
		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
		for (String postProcessorName : orderedPostProcessorNames) {
			orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

		//执行普通的 BeanFactoryPostProcessor的 postProcessBeanFactory方法
		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
		for (String postProcessorName : nonOrderedPostProcessorNames) {
      //挑出符合 BeanFactoryPostProcessor类型的
			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

		// Clear cached merged bean definitions since the post-processors might have
		// modified the original metadata, e.g. replacing placeholders in values...
		beanFactory.clearMetadataCache();
	}
```

#### 【附注】ConfigurationClassPostProcessor扫描、注册的详细流程

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
		List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
		//获取 BeanDefinitionMap 中的 BeanDefinitionNameS
		String[] candidateNames = registry.getBeanDefinitionNames();

		for (String beanName : candidateNames) {
			BeanDefinition beanDef = registry.getBeanDefinition(beanName);
			if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
				if (logger.isDebugEnabled()) {
					logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
				}
			}
      //检查该 DB 关联的Class是否是一个配置类
			else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
				configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
			}
		}
		// 如果没有配置类就直接返回
		if (configCandidates.isEmpty()) {
			return;
		}
		// 根据 @Order 对配置类递归排序
		configCandidates.sort((bd1, bd2) -> {
			int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
			int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
			return Integer.compare(i1, i2);
		});

		// Detect any custom bean name generation strategy supplied through the enclosing application context
		SingletonBeanRegistry sbr = null;
		if (registry instanceof SingletonBeanRegistry) {
			sbr = (SingletonBeanRegistry) registry;
			if (!this.localBeanNameGeneratorSet) {
				BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
						AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);
				if (generator != null) {
					this.componentScanBeanNameGenerator = generator;
					this.importBeanNameGenerator = generator;
				}
			}
		}

		if (this.environment == null) {
			this.environment = new StandardEnvironment();
		}

		// Parse each @Configuration class
		ConfigurationClassParser parser = new ConfigurationClassParser(
				this.metadataReaderFactory, this.problemReporter, this.environment,
				this.resourceLoader, this.componentScanBeanNameGenerator, registry);

		Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
		Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
		do {
			StartupStep processConfig = this.applicationStartup.start("spring.context.config-classes.parse");
			//执行所有配置类的解析，1.先构建 ConfigurationClass实例
      parser.parse(candidates);
			parser.validate();

			Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
			configClasses.removeAll(alreadyParsed);

			// Read the model and create bean definitions based on its content
			if (this.reader == null) {
				this.reader = new ConfigurationClassBeanDefinitionReader(
						registry, this.sourceExtractor, this.resourceLoader, this.environment,
						this.importBeanNameGenerator, parser.getImportRegistry());
			}
			this.reader.loadBeanDefinitions(configClasses);
			alreadyParsed.addAll(configClasses);
			processConfig.tag("classCount", () -> String.valueOf(configClasses.size())).end();

			candidates.clear();
			if (registry.getBeanDefinitionCount() > candidateNames.length) {
				String[] newCandidateNames = registry.getBeanDefinitionNames();
				Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
				Set<String> alreadyParsedClasses = new HashSet<>();
				for (ConfigurationClass configurationClass : alreadyParsed) {
					alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
				}
				for (String candidateName : newCandidateNames) {
					if (!oldCandidateNames.contains(candidateName)) {
						BeanDefinition bd = registry.getBeanDefinition(candidateName);
						if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
								!alreadyParsedClasses.contains(bd.getBeanClassName())) {
							candidates.add(new BeanDefinitionHolder(bd, candidateName));
						}
					}
				}
				candidateNames = newCandidateNames;
			}
		}
		while (!candidates.isEmpty());

		// Register the ImportRegistry as a bean in order to support ImportAware @Configuration classes
		if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
			sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
		}

		if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
			// Clear cache in externally provided MetadataReaderFactory; this is a no-op
			// for a shared cache since it'll be cleared by the ApplicationContext.
			((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
		}
	}
```

![image-20220822111050012](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220822111050012.png)

## IOC扩展手段







































