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



#  bean的实例化流程

0. 实例化IOC容器，执行refresh()
   1. 扫描包,生成beanDef -->beanDefMap
1. 遍历beanDefinitionNames，逐个处理解析的beanDef
   1. 验证，要求beanDef为非抽象、单例、非懒加载。否则跳过
   2. 判断是否为FactoryBean，如果是就做特殊处理，否则直接getBean()。
2. doGetBean()
   1. 判断该beanName是否已被手动创建
   2. 若该beanName处于正在创建的原型bean集合之中，就抛异常
   3. 若存在父容器就调用 parentBeanFactory.getBean(），否则
   4. 经历一系列的核查，包括是否@DependsOn
3. `getSingleton(beanName, () -> {}` ->createBean()
   1. 获取beanName的class类型（反射）
   2. 尝试代理创建Bean，否则
   3. 创建beanName的class类型的java对象实例【通过推断构造方法实现（反射）】

零 spring容器对象 AnnotationConfigApplicationContext 被实例化了 然后执行refresh方法进行初始化 * 1、扫描包--beanDefine --put bdmap * 21 变量这个bdmap 依次获取 beanDefinition对象 * 3、去解析---吧beandefinition对象当中描述bean的信息得到 * 4、validate 验证是否单例 是否抽象 是否factoryBean 是否被创建。。。 5、得到class的对象 *6、推断构造方法 一个类可以提供多个构造方法 spring自实例化bean的时候需要用哪个构造方法 * 7、得到一个构造方法--通过反射然后直接实例化这个对象(这个时候还不是一个bean) 8、做一个循环依赖的判断然后对喜欢依赖提供支持 * 9、合并beanDefinition(不需要关心) * 10、判断当前容器是否需要完成自动注入 程序员可以扩展的 * 11、如果需要完成注入 完成属性填充---自动注入 * 12、执行部分的aware接口(很多的Aware 这里不会全部执行 只会执行一部分) * 13、(BeanPostProcessor before)执行部分生命周期的初始化回调(注解版本) 执行部分Aware接口的回调 * 14、执行接口版的生命周期回调 * 15、(BeanPostProcessor after) 完成的事件发布 完成aop的代理 */