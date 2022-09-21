# IOC体系的核心类

## "容器族"

### BeanFactory

BeanFactory会将 Bean定义配置成Bean，并提供对Bean的访问。



## ”配置族“

### BeanFactoryPostProcessor

![image-20220820133901209](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220820133901209.png)

允许修改IOC中**已经存在**的beanDef的属性，进而更新 Bean工厂中Bean的属性值。<u>但不能与Bean 实例交互，也不能向beanDefMap中添加beanDef。</u>这样做可能会导致过早的 bean 实例化，违反容器规范并导致意外的副作用

### BeanDefinitionRegistryPostProcessor

在 BeanFactoryPostProcessor 方法之前执行，能对配置类扫描生成的BeanDef进行更新并注册到beanDefMap里。也可以通过该类对beanDefMap中的beanDef进行增加和删除。

### ConfigurationClassPostProcessor

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


### ImportBeanDefinitionRegistrar

与以上三者绝然不同，它支持对 BeanDefinition 的新增和移除，支持扩展，强大！,



#  Spring生命周期浅析

整个Spring的生命周期，大概分为以下几个阶段：

1. 初始化bean容器
2. 将Spring内置的必须的环境级别的class文件转换为bd
3. 初始化bean工厂，设置默认值
4. 向BeanFactory中注册一些Spring内置的Bean后置处理器
5. 执行内置的`BeanFactoryPostProcessor`，按要求扫描bean组件，将其解析成 `BeanDefinition`并存入map
6. 执行自定义的 `BeanFactoryPostProcessor`
7. 注册所有的`BeanPostProcessor`到容器内部！

8. 初始化国际化资源

9. 初始化事件资源

10. 实例化class

11. 为class实例填充属性（自动注入）

12. 回调`BeanPostProcessors.postProcessBeforeInitialization`方法

13. 调用bean的初始化方法

14. 回调`BeanPostProcessors.postProcessAfterInitialization`方法



# IOC初始化流程 {5.3.22}

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
    this.refresh(); // 刷新IOC容器，扩展Bean、更改Bean定义，创建bean实例
    }
```

## 1. `this();` //创建IOC容器

1. `AnnotationConfigApplicationContext#AnnotationConfigApplicationContext(clazz):`创建IOC容器刷新初始化配置
   1. `GenericApplicationContext#GenericApplicationContext():` 调用父类的无参构造，初始化`DefaultResourceLoader ` 、**`DefaultListableBeanFactory`**
   2. 初始化`AnnotatedBeanDefinitionReader `，该过程会调用`AnnotationConfigUtils#registerAnnotationConfigProcessors(BeanDefinitionRegistry)`将默认的beanDef注册到beanDefMap中，对应类的先后顺序为**1.** ConfigurationClassPostProcessor **2.**DefaultEventListenerFactory **3.**  EventListenerMethodProcessor **4.** AutowiredAnnotationBeanPostProcessor **5.** CommonAnnotationBeanPostProcessor
   3. 初始化`ClassPathBeanDefinitionScanner`

## 2. `this.register(componentClasses);` // 注册指定(配置)类

1. 可传入clazz数组，由`AnnotatedBeanDefinitionReader#register ` 循环调用`AnnotatedBeanDefinitionReader#doRegisterBean(clazzs);`来注册数组中所有的类
   1. 为目标类生成AnnotatedGenericBeanDefinition实例[`agbd`]，当clazz全类名为`null`或者该类被`Conditional`注解标注后，会跳过注册。
   2. 如果没有跳过，就会为`agbd`设置 beanDefinition属性，然后由`AnnotationConfigUtils.processCommonDefinitionAnnotations(agbd)`集中处理该类的注解，并为abd赋值，包括`@Lazy`、`@Primary`、`@DependsOn`、`@Role`、`@Description`。最后判断处理qualifiers、customizers。
   3. 根据agdb和beanName生成BeanDefinitionHolder，并调用`DefaultListableBeanFactory#registerBeanDefinition()`完成注册。该过程中会依次判断该BeanDefinition是否属于AbstractBeanDefinition、以及beanDefinitionMap中是否已经存在该BeanDefinition、最后检测该bean是否已经开始创建了即`AbstractBeanFactory#alreadyCreated`这个Set中是否为空。完成注册后会将该bean的别名注册到BeanDefinitionRegistry中**[如果有]**。

## 3.` this.refresh(); `// 刷新IOC容器

### 3.1 `prepareRefresh();`

刷新容器的前置准备，包括设置容器的启动时间、设置活跃和关闭状态、初始化占位符属性资源、获取环境对象并校验必须属性是否可解析、初始化earlyApplicationListeners、earlyApplicationEvents。

### 3.2 `obtainFreshBeanFactory();` 

获取更新的ConfigurableListableBeanFactory，设置序列化的beanFactoryId,

### 3.3 `prepareBeanFactory(beanFactory);` 

为beanFactory配置属性。包括Bean的类加载器、bean的属性编辑注册器、Bean的表达式解析器、Bean的后置处理器【ApplicationContextAwareProcessor、ApplicationListenerDetector、LoadTimeWeaverAwareProcessor】、最后将4个默认的环境bean environment、systemProperties、systemEnvironment、applicationStartup注册到单例池中。

### 3.4 `postProcessBeanFactory(beanFactory);` 

doNothing，留给子类扩展，根据自定义的ConfigurableListableBeanFactory来加载所有指定的beanDefinition。

### 3.5 `invokeBeanFactoryPostProcessors(beanFactory);`

调用执行所有的bean工厂的后置处理器。根据配置类、及自定义扩展执行扫描，找出所有beanFactory后置处理器，并且调用它们的实现来更新Bean定义，最后注册到BeanDefinitionMap中。简单说就是，解析配置类、更新beanDef和此处采用的是分批分步执行的策略：

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-08-21_14-03-42.png)

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

**【注意】: 除去自定义扩展的postProcessor，其他所有的postProcessor都会在记录之前直接执行getBean()提前实例化**

上述有三处invokeBeanDefinitionRegistryPostProcessors(),

### 3.6 `registerBeanPostProcessors(beanFactory);` 

注册Bean的后置处理器，只注册不执行。

### 3.7 `initMessageSource();` 

初始化message源，国际化支持。

### 3.8 `initApplicationEventMulticaster();` 

初始化事件多路广播器。

### 3.9 `onRefresh();` 

doNothing，留给子类扩展初始化其他bean。

### 3.10 `registerListeners();` 

将所有的监听器注册到广播器中。

### 3.11 `finishBeanFactoryInitialization(beanFactory);` 

实例化剩余的非Lazy单例bean。

### 3.12 `finishRefresh();` 

刷新上下文，初始化生命周期。

### 3.13 `resetCommonCaches();`

清空通用缓存。包括：

```java
// 反射的缓存
ConcurrentReferenceHashMap<Class<?>, Method[]> declaredMethodsCache;
ConcurrentReferenceHashMap<Class<?>, Field[]> declaredFieldsCache;
// 注解的缓存
ConcurrentReferenceHashMap<AnnotationFilter, Cache> standardRepeatablesCache;
ConcurrentReferenceHashMap<AnnotationFilter, Cache> noRepeatablesCache;
ConcurrentReferenceHashMap<AnnotatedElement, Annotation[]> declaredAnnotationCache;
ConcurrentReferenceHashMap<Class<?>, Method[]> baseTypeMethodsCache;
// 可解析类型的缓存
ConcurrentReferenceHashMap<ResolvableType, ResolvableType> cache;
ConcurrentReferenceHashMap<Type, Type> cache;
// 清空指定的类加载器
```



### 【附注】ConfigurationClassPostProcessor配置类处理的详细流程

在3.5中有一些极为关键的方法，不得不知：

#### `invokeBeanDefinitionRegistryPostProcessors(...)`

该方法最终调用的是`ConfigurationClassPostProcessor#processConfigBeanDefinitions()`；它的核心作用是：

1. 检测配置类
2. 解析配置类，并注册新检测到的beanDef。并循环此步骤直至没有未解析配置类为止。

**流程：**

##### processConfigBeanDefinitions();集中处理配置类

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    // 存储所有的配置类的beanDef
		List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
		//获取 beanFactory中beanDefinitionNames的所有元素
		String[] candidateNames = registry.getBeanDefinitionNames();
		for (String beanName : candidateNames) {
			BeanDefinition beanDef = registry.getBeanDefinition(beanName);
      // 该beanDef已被解析过，在配置类检测时会为该beanDef添加此属性[属性值为full或者lite的即为配置类]
			if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
				if (logger.isDebugEnabled()) {
					logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
				}
			}
      //对beanDef进行配置类检测
			else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
				configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
			}
		}
		// 如果没有配置类就直接返回
		if (configCandidates.isEmpty()) {
			return;
		}
		// 根据@Order对配置类递归排序
		configCandidates.sort((bd1, bd2) -> {
			int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
			int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
			return Integer.compare(i1, i2);
		});

		// Detect any custom bean name generation strategy supplied through the enclosing application context
    //如果当前beanFactory属于单例的，就获取beanName的生成器
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
    // 开始解析每一个配置类 @Configuration 
    // 准备配置类解析器ConfigurationClassParser
		ConfigurationClassParser parser = new ConfigurationClassParser(
				this.metadataReaderFactory, this.problemReporter, this.environment,
				this.resourceLoader, this.componentScanBeanNameGenerator, registry);
    //保存将要被解析的beanDef
		Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
    //保存已被解析的配置类，是类
		Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
		do {
			StartupStep processConfig = this.applicationStartup.start("spring.context.config-classes.parse");
			//开始执行所有配置类的解析
      parser.parse(candidates);
      //校验规则：full型的配置类不能为final；@Bean修饰的方法必须是可覆盖的。因为full型需要cglib代理， @Bean所修饰的方法也有一套约束规则，下面详细讲
			// 是否需要代理是根据@Scope 注解指定的，默认都是不代理
			parser.validate();
      //保存此次解析出的所有配置类
			Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
      //移除解析过的
			configClasses.removeAll(alreadyParsed);
			// Read the model and create bean definitions based on its content
			if (this.reader == null) {
				this.reader = new ConfigurationClassBeanDefinitionReader(
						registry, this.sourceExtractor, this.resourceLoader, this.environment,
						this.importBeanNameGenerator, parser.getImportRegistry());
			}
      //注册当前新解析的所有的bean
			this.reader.loadBeanDefinitions(configClasses);
      //记录为已解析
			alreadyParsed.addAll(configClasses);
			processConfig.tag("classCount", () -> String.valueOf(configClasses.size())).end();
			//清空解析任务清单
			candidates.clear();
      //是否有新的beanDef被注册
			if (registry.getBeanDefinitionCount() > candidateNames.length) {
        //存有当前所有的beanDef
				String[] newCandidateNames = registry.getBeanDefinitionNames();
        //旧的解析任务清单
				Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
				Set<String> alreadyParsedClasses = new HashSet<>();
				for (ConfigurationClass configurationClass : alreadyParsed) {
          //取出所有已被解析过的配置类的名称添加到alreadyParsedClasses
					alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
				}
				for (String candidateName : newCandidateNames) {
					if (!oldCandidateNames.contains(candidateName)) {
						BeanDefinition bd = registry.getBeanDefinition(candidateName);
            //如果当前beanDef是新来的，并且是一个没被解析过的配置类型的beanDef,就把它添加到解析任务中
						if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
								!alreadyParsedClasses.contains(bd.getBeanClassName())) {
							candidates.add(new BeanDefinitionHolder(bd, candidateName));
						}
					}
				}
				candidateNames = newCandidateNames;
			}
		}
		while (!candidates.isEmpty());//只要解析任务清单不为空，就一致循环下去，实现解析所有的配置类
    // 将ImportRegistry注册为bean，以支持ImportAware @Configuration 类
		if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
			sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
		}
		//清除缓存
		if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
			((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
		}
	}
```

##### checkConfigurationClassCandidate();判断是否为配置类

什么是配置类呢？配置类在代码中的体现是；该beanDef存在一个名为configurationClass的属性，且属性值必须为：

+ **full：** 即类被 @Configuration 注解修饰并且 proxyBeanMethods属性为true (默认为 true)
+ **lite：** 即类被 @Component、@ComponentScan、@Import、@ImportResource 修饰的类 或者 类中有被@Bean修饰的方法。

通俗说就是：允许为其他类或者自己生成beanDef的类、允许将指定文件解析成可用的propertySource的类

```java
public static boolean checkConfigurationClassCandidate(BeanDefinition beanDef, MetadataReaderFactory metadataReaderFactory) {
   String className = beanDef.getBeanClassName();
   if (className == null || beanDef.getFactoryMethodName() != null) {
      return false;
   }
   AnnotationMetadata metadata;
   //该beanDef关联的Class是否有被注解
   if (beanDef instanceof AnnotatedBeanDefinition &&
         className.equals(((AnnotatedBeanDefinition) beanDef).getMetadata().getClassName())) {
      metadata = ((AnnotatedBeanDefinition) beanDef).getMetadata();
   }
   else if (beanDef instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) beanDef).hasBeanClass()) {
      // Check already loaded Class if present...
      // since we possibly can't even load the class file for this Class.
      // 如果该beanClass属于以下4个类型就不做配置类处理
      Class<?> beanClass = ((AbstractBeanDefinition) beanDef).getBeanClass();
      if (BeanFactoryPostProcessor.class.isAssignableFrom(beanClass) ||
            BeanPostProcessor.class.isAssignableFrom(beanClass) ||
            AopInfrastructureBean.class.isAssignableFrom(beanClass) ||
            EventListenerFactory.class.isAssignableFrom(beanClass)) {
         return false;
      }
      metadata = AnnotationMetadata.introspect(beanClass);
   }
   else {
      try {
      	 //解析注解元数据
         MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(className);
         metadata = metadataReader.getAnnotationMetadata();
      }
      catch (IOException ex) {
         if (logger.isDebugEnabled()) {
            logger.debug("Could not find class file for introspecting configuration annotations: " +
                  className, ex);
         }
         return false;
      }
   }
   //获取beanClass上的Configuration注解的属性
   Map<String, Object> config = metadata.getAnnotationAttributes(Configuration.class.getName());
   if (config != null && !Boolean.FALSE.equals(config.get("proxyBeanMethods"))) {
      //如果被@Configuration标注且proxyBeanMethods属性为true，就设置为full
      beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_FULL);
   }
   else if (config != null || isConfigurationCandidate(metadata)) {
     	//如果被@Configuration标注且isConfigurationCandidate(metadata)为true，就设置为lite
      beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_LITE);
   }
   else {
      return false;
   }
   // 按照@Order排序
   Integer order = getOrder(metadata);
   if (order != null) {
      beanDef.setAttribute(ORDER_ATTRIBUTE, order);
   }
   return true;
}
```

##### isConfigurationCandidate();能否充当配置

名字自己取的，方便理解。

```java
public static boolean isConfigurationCandidate(AnnotationMetadata metadata) {
   // 不考虑接口和注解
   if (metadata.isInterface()) {
      return false;
   }
   // 是否被@ComponentScan、@Component、@Import、@ImportResource
   for (String indicator : candidateIndicators) {
      if (metadata.isAnnotated(indicator)) {
         return true;
      }
   }
   // beanClass是否有@Bean标注的方法
   return hasBeanMethods(metadata);
}
```

##### parse();执行配置类解析

最终会调用`ConfigurationClassParser#doProcessConfigurationClass`来解析配置类

**解析流程：**

1. 解析@Component：【总结】先解析该配置类自身及其内部成员类
   1. 判断该配置类是否有lite型的内部成员类
   2. 如果有就循环、递归调用 `doProcessConfigurationClass()` 
   3. 解析后会生成ConfigurationClass，暂存到`CongigurationClassParser.configurationClasses`中
2. 解析@PropertySource：将指向的文件内容存入 `environment.propertySources`
3. 解析@ComponentScan、@ComponentScans
   1. 解析@ComponentScans注解的属性并封装成componentscan 对象
   2. 获取类路径扫描器 ，设置扫描器属性，默认有**@component**注解类型的包含过滤器；设置@ComponentScan的属性，将其包含和排除过滤拷贝到扫描器中。
   3. 由扫描器通过类路径对指定路径下的所有类进行判定。判定标准：在符合包含过滤的前提下基本就可以通过。判定通过后会将具体的、独立的类对应的sgBeanDef添加到候选清单，并集中返回。
   4. 拿到任务清单后，会循环为beanDef赋值，在检查该beanDef在beanDefMap不存在时才会执行注册
   5. 遍历扫描生成的beanDef，若为配置类型，就递归解析
4. 解析@Import
   1. 依次解析@Import、@ImportSelector、ImportBeanDefinitionRegistrar实现。
5. 解析@ImportResource
   1. 直接保存到ConfigurationClass中
6. 解析@Bean
   1. 直接保存到ConfigurationClass中
7. 递归处理其他接口默认方法中的BeanMethod
8. 如果当前类存在不属于jdk中且未解析过的父类，就返回父类对父类进行解析。

```java
protected final SourceClass doProcessConfigurationClass(
      ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
      throws IOException {
	 //处理所有的@Component
   if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
      //执行
      processMemberClasses(configClass, sourceClass, filter);
   }
   // 处理所有的@PropertySource
   for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
         sourceClass.getMetadata(), PropertySources.class,
         org.springframework.context.annotation.PropertySource.class)) {
      if (this.environment instanceof ConfigurableEnvironment) {
         //执行
         processPropertySource(propertySource);
      }
      else {
         logger.info("Ignoring @PropertySource annotation on [" + sourceClass.getMetadata().getClassName() +
               "]. Reason: Environment must implement ConfigurableEnvironment");
      }
   }
   //处理所有的@ComponentScan
   //解析@ComponentScans注解的属性并封装成componentscan 对象
   Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
         sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
   if (!componentScans.isEmpty() &&
         !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
      for (AnnotationAttributes componentScan : componentScans) {
         // 执行解析
         Set<BeanDefinitionHolder> scannedBeanDefinitions =
               this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
         // 遍历扫描生成的beanDef
         for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
            BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
            if (bdCand == null) {
               bdCand = holder.getBeanDefinition();
            }
            //如果扫描得到的beanDef是配置类型的，就执行递归解析
            if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
               parse(bdCand.getBeanClassName(), holder.getBeanName());
            }
         }
      }
   }
   //处理所有的@Import
   processImports(configClass, sourceClass, getImports(sourceClass), filter, true);
   //处理所有的@ImportResource
   AnnotationAttributes importResource =
         AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
   if (importResource != null) {
      String[] resources = importResource.getStringArray("locations");
      Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
      for (String resource : resources) {
         String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
         //直接存入ConfigurationClass#importedResources中
         configClass.addImportedResource(resolvedResource, readerClass);
      }
   }
   //处理所有@Bean
   Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
   for (MethodMetadata methodMetadata : beanMethods) {
      //直接存入ConfigurationClass#beanMethods
      configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
   }
   //递归处理其他接口默认方法中的BeanMethod
   processInterfaces(configClass, sourceClass);
   //处理父类
   if (sourceClass.getMetadata().hasSuperClass()) {
      String superclass = sourceClass.getMetadata().getSuperClassName();
      if (superclass != null && !superclass.startsWith("java") &&
            !this.knownSuperclasses.containsKey(superclass)) {
         this.knownSuperclasses.put(superclass, configClass);
         // Superclass found, return its annotation metadata and recurse
         return sourceClass.getSuperClass();
      }
   }
   return null;
}
```

###### processMemberClasses();解析@Component

```java
private void processMemberClasses(ConfigurationClass configClass, SourceClass sourceClass,
			Predicate<String> filter) throws IOException {
	  //拿到内部类
		Collection<SourceClass> memberClasses = sourceClass.getMemberClasses();
		if (!memberClasses.isEmpty()) {
			List<SourceClass> candidates = new ArrayList<>(memberClasses.size());
			for (SourceClass memberClass : memberClasses) {
        //判断此内部类能否充当lite型配置类
				if (ConfigurationClassUtils.isConfigurationCandidate(memberClass.getMetadata()) &&
	!memberClass.getMetadata().getClassName().equals(configClass.getMetadata().getClassName())) {
          //将此内部类添加到解析任务清单
					candidates.add(memberClass);
				}
			}
			OrderComparator.sort(candidates);
			for (SourceClass candidate : candidates) {
        // importStack用来缓存已经解析过的内部类，这里处理循环引入问题。
				if (this.importStack.contains(configClass)) {
					this.problemReporter.error(new CircularImportProblem(configClass, this.importStack));
				}
				else {
          // 解析前入栈
					this.importStack.push(configClass);
					try {
            // 递归解析新发现的配置类
						processConfigurationClass(candidate.asConfigClass(configClass), filter);
					}
					finally {
            // 解析完出栈，保证只解析一次
						this.importStack.pop();
					}
				}
			}
		}
	}
```



###### processPropertySource();解析@PropertySource

```java
private void processPropertySource(AnnotationAttributes propertySource) throws IOException {
  //获取@PropertySource的指定属性值
  String name = propertySource.getString("name");
   if (!StringUtils.hasLength(name)) {
      name = null;
   }
   String encoding = propertySource.getString("encoding");
   if (!StringUtils.hasLength(encoding)) {
      encoding = null;
   }
   //获取指向的文件路径
   String[] locations = propertySource.getStringArray("value");
   Assert.isTrue(locations.length > 0, "At least one @PropertySource(value) location is required");
   boolean ignoreResourceNotFound = propertySource.getBoolean("ignoreResourceNotFound");
   Class<? extends PropertySourceFactory> factoryClass = propertySource.getClass("factory");
   PropertySourceFactory factory = (factoryClass == PropertySourceFactory.class ?
         DEFAULT_PROPERTY_SOURCE_FACTORY : BeanUtils.instantiateClass(factoryClass));
   //遍历文件路径,解析指向的文件并将内容存储到 environment.propertySources 中
   for (String location : locations) {
      try {
         String resolvedLocation = this.environment.resolveRequiredPlaceholders(location);
         Resource resource = this.resourceLoader.getResource(resolvedLocation);
         addPropertySource(factory.createPropertySource(name, new EncodedResource(resource, encoding)));
      }
      catch (IllegalArgumentException | FileNotFoundException | UnknownHostException | SocketException ex) {
         // Placeholders not resolvable or resource not found when trying to open it
         if (ignoreResourceNotFound) {
            if (logger.isInfoEnabled()) {
               logger.info("Properties location [" + location + "] not resolvable: " + ex.getMessage());
            }
         }
         else {
            throw ex;
         }
      }
   }
}
```



###### parse();解析@ComponentScan

作用：递归解析指定路径下的lite型配置类

```java
public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, String declaringClass) {
  //获取类路径扫描器 
  ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(this.registry,
         componentScan.getBoolean("useDefaultFilters"), this.environment, this.resourceLoader);
   //设置扫描器属性
   Class<? extends BeanNameGenerator> generatorClass = componentScan.getClass("nameGenerator");
   boolean useInheritedGenerator = (BeanNameGenerator.class == generatorClass);
   scanner.setBeanNameGenerator(useInheritedGenerator ? this.beanNameGenerator :
         BeanUtils.instantiateClass(generatorClass));
   ScopedProxyMode scopedProxyMode = componentScan.getEnum("scopedProxy");
   if (scopedProxyMode != ScopedProxyMode.DEFAULT) {
      scanner.setScopedProxyMode(scopedProxyMode);
   }
   else {
      Class<? extends ScopeMetadataResolver> resolverClass = componentScan.getClass("scopeResolver");
      scanner.setScopeMetadataResolver(BeanUtils.instantiateClass(resolverClass));
   }
   scanner.setResourcePattern(componentScan.getString("resourcePattern"));
   //设置包含过滤
   for (AnnotationAttributes includeFilterAttributes : componentScan.getAnnotationArray("includeFilters")) {
      List<TypeFilter> typeFilters = TypeFilterUtils.createTypeFiltersFor(includeFilterAttributes, this.environment,
            this.resourceLoader, this.registry);
      for (TypeFilter typeFilter : typeFilters) {
         scanner.addIncludeFilter(typeFilter);
      }
   }
   //设置排除过滤
   for (AnnotationAttributes excludeFilterAttributes : componentScan.getAnnotationArray("excludeFilters")) {
      List<TypeFilter> typeFilters = TypeFilterUtils.createTypeFiltersFor(excludeFilterAttributes, this.environment,
         this.resourceLoader, this.registry);
      for (TypeFilter typeFilter : typeFilters) {
         scanner.addExcludeFilter(typeFilter);
      }
   }
   //设置LazyInit
   boolean lazyInit = componentScan.getBoolean("lazyInit");
   if (lazyInit) {
      scanner.getBeanDefinitionDefaults().setLazyInit(true);
   }
   //存储要扫描的路径
   Set<String> basePackages = new LinkedHashSet<>();
   //获取扫描路径数组
   String[] basePackagesArray = componentScan.getStringArray("basePackages");
   for (String pkg : basePackagesArray) {
      String[] tokenized = StringUtils.tokenizeToStringArray(this.environment.resolvePlaceholders(pkg),
            ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
      Collections.addAll(basePackages, tokenized);
   }
   for (Class<?> clazz : componentScan.getClassArray("basePackageClasses")) {
      basePackages.add(ClassUtils.getPackageName(clazz));
   }
   if (basePackages.isEmpty()) {
      basePackages.add(ClassUtils.getPackageName(declaringClass));
   }
   scanner.addExcludeFilter(new AbstractTypeHierarchyTraversingFilter(false, false) {
      @Override
      protected boolean matchClassName(String className) {
         return declaringClass.equals(className);
      }
   });
   //执行扫描
   return scanner.doScan(StringUtils.toStringArray(basePackages));
}
```

###### doScan();执行@ComponentScan的扫描

```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
   Assert.notEmpty(basePackages, "At least one base package must be specified");
   Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
   for (String basePackage : basePackages) {
      //执行扫描，并生成指定路径下所有通过核实的类的beanDef
      Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
      for (BeanDefinition candidate : candidates) {
         ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
         //为该beanDef设置属性值
         candidate.setScope(scopeMetadata.getScopeName());
         String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
         if (candidate instanceof AbstractBeanDefinition) {
            postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
         }
         if (candidate instanceof AnnotatedBeanDefinition) {
            //为beanDef设置默认的属性值，模板方法
            AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
         }
         //检查是否允许注册，beanDefinitionMap中是否已存在该beanName
         if (checkCandidate(beanName, candidate)) {
            BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
            definitionHolder =
                  AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
            beanDefinitions.add(definitionHolder);
            //执行注册
            registerBeanDefinition(definitionHolder, this.registry);
         }
      }
   }
   return beanDefinitions;
}
```

###### @ComponentScan扫描细节

由findCandidateComponents(basePackage);调用

```java
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
   Set<BeanDefinition> candidates = new LinkedHashSet<>();
   try {
      String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
            resolveBasePackage(basePackage) + '/' + this.resourcePattern;
      //持有该包下所有类的物理文件路径
      Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
      boolean traceEnabled = logger.isTraceEnabled();
      boolean debugEnabled = logger.isDebugEnabled();
      // resource  H:\Study\spring_study\target\classes\entity\Person.class
      for (Resource resource : resources) {
         if (traceEnabled) {
            logger.trace("Scanning " + resource);
         }
         try {
            MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
           //根据当前类的元数据判断是否可以作为候选 
           if (isCandidateComponent(metadataReader)) {
               //根据当前类的元数据创建beanDef
               ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
               sbd.setSource(resource);
               //判断当前beanDef能否作为候选
               //1.该beanDef关联的clazz是否独立(无属性对象)且是否是具体的类(非接口、非抽象)
               //2.或者，是一个被@Lookup标注的抽象类
               if (isCandidateComponent(sbd)) {
                  if (debugEnabled) {
                     logger.debug("Identified candidate component class: " + resource);
                  }
                  //正式成为候选，并保存
                  candidates.add(sbd);
               }
               else {
                  if (debugEnabled) {
                     logger.debug("Ignored because not a concrete top-level class: " + resource);
                  }
               }
            }
            else {
               if (traceEnabled) {
                  logger.trace("Ignored because not matching any filter: " + resource);
               }
            }
         }
         catch (FileNotFoundException ex) {
            if (traceEnabled) {
               logger.trace("Ignored non-readable " + resource + ": " + ex.getMessage());
            }
         }
         catch (Throwable ex) {
            throw new BeanDefinitionStoreException(
                  "Failed to read candidate component class: " + resource, ex);
         }
      }
   }
   catch (IOException ex) {
      throw new BeanDefinitionStoreException("I/O failure during classpath scanning", ex);
   }
   return candidates;
}
```

```java
protected boolean isCandidateComponent(MetadataReader metadataReader) throws IOException {
   // metadataReader：指向当前候选
   // getMetadataReaderFactory()：存有指定包路径下所有的类信息
   for (TypeFilter tf : this.excludeFilters) {
      //排除过滤：如果当前候选在 getMetadataReaderFactory() 之列，就直接拒绝它。
      if (tf.match(metadataReader, getMetadataReaderFactory())) {
         return false;
      }
   }
   for (TypeFilter tf : this.includeFilters) {、
      //许可过滤：如果当前候选在 getMetadataReaderFactory() 之列，就尝试添加它。
      if (tf.match(metadataReader, getMetadataReaderFactory())) {
         // 是否存在指定的注解，默认有@Component
         return isConditionMatch(metadataReader);
      }
   }
   return false;
}
```

###### @ComponentScan允许添加的条件细要

由 isConditionMatch()；调用

```java
public boolean shouldSkip(@Nullable AnnotatedTypeMetadata metadata, @Nullable ConfigurationPhase phase) {
   // metadata : 候选 //此处phase无值：null
   // 当候选不存在或没有被@Conditional标注，就允许添加
   if (metadata == null || !metadata.isAnnotated(Conditional.class.getName())) {
      return false;
   }
   if (phase == null) {
      if (metadata instanceof AnnotationMetadata &&
            ConfigurationClassUtils.isConfigurationCandidate((AnnotationMetadata) metadata)) {
         //如果候选是lite型配置类，就将其作为配置类，再次解析判断
         return shouldSkip(metadata, ConfigurationPhase.PARSE_CONFIGURATION);
      }
      //如果不是配置类，就再次判断是否要为这个候选注册beanDef
      return shouldSkip(metadata, ConfigurationPhase.REGISTER_BEAN);
   }
   //存储所有的自定义Condition实现
   List<Condition> conditions = new ArrayList<>();
   //封装@Conditional注解
   for (String[] conditionClasses : getConditionClasses(metadata)) {
      for (String conditionClass : conditionClasses) {
         Condition condition = getCondition(conditionClass, this.context.getClassLoader());
         conditions.add(condition);
      }
   }
   AnnotationAwareOrderComparator.sort(conditions);
   for (Condition condition : conditions) {
      ConfigurationPhase requiredPhase = null;
      //如果候选实现的是ConfigurationCondition接口,就获取自定义的标志
      if (condition instanceof ConfigurationCondition) {
         requiredPhase = ((ConfigurationCondition) condition).getConfigurationPhase();
      }
      //基本上就是false，即允许添加
      //1.macthes(.,.)默认是true，即后半部分是false
      //2.如果候选是Condition实现，则requiredPhase为null，而phase有被Spring赋值 --> false
      //3.如果候选是ConfigurationCondition实现，则requiredPhase不为null --> false
      if ((requiredPhase == null || requiredPhase == phase) && !condition.matches(this.context, metadata)) {
         return true;
      }
   }
   return false;
}
```



###### processImports(); 解析@Import【未Debug】

+ @Import ： 可以通过 @Import(XXX.class) 的方式，将指定的类注册到容器中
+ ImportSelector : Spring会将 ImportSelector#selectImports 方法返回的内容通过反射加载到容器中
+ ImportBeanDefinitionRegistrar ： 可以通过 registerBeanDefinitions 方法声明BeanDefinition 并自己注册到Spring容器中 

这里解析的ImportSelector、ImportBeanDefinitionRegistrar 都是通过 @Import 注解引入的。如果不是通过 @Import 引入(比如直接通过@Component 将ImportSelector、ImportBeanDefinitionRegistrar 注入)的类则不会被解析。

```java
private void processImports(ConfigurationClass configClass, SourceClass currentSourceClass,
      Collection<SourceClass> importCandidates, Predicate<String> exclusionFilter,
      boolean checkForCircularImports) {
   //如果未使用@Import就直接返回
   if (importCandidates.isEmpty()) {
      return;
   }
   //检测循环引入
   if (checkForCircularImports && isChainedImportOnStack(configClass)) {
      this.problemReporter.error(new CircularImportProblem(configClass, this.importStack));
   }
   else {
      this.importStack.push(configClass);
      try {
         for (SourceClass candidate : importCandidates) {
            //判断是否是ImportSelector类型。
            if (candidate.isAssignable(ImportSelector.class)) {
               Class<?> candidateClass = candidate.loadClass();
               ImportSelector selector = ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class,
                     this.environment, this.resourceLoader, this.registry);
               Predicate<String> selectorFilter = selector.getExclusionFilter();
               if (selectorFilter != null) {
                  exclusionFilter = exclusionFilter.or(selectorFilter);
               }
               if (selector instanceof DeferredImportSelector) {
                  this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);
               }
               else {
                  // 调用ImportSelector.selectImports获取所有需要注册的类名
                  String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
                  Collection<SourceClass> importSourceClasses = asSourceClasses(importClassNames, exclusionFilter); 
                  //递归解析
                  processImports(configClass, currentSourceClass, importSourceClasses, exclusionFilter, false);
               }
            }
            //如果是ImportBeanDefinitionRegistrar类型
            else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
               //则直接加载注册指定的clazz
               Class<?> candidateClass = candidate.loadClass();
               ImportBeanDefinitionRegistrar registrar =
                     ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class,
                           this.environment, this.resourceLoader, this.registry);
               configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
            }
            else {
               // Candidate class not an ImportSelector or ImportBeanDefinitionRegistrar ->
               // process it as an @Configuration class
               this.importStack.registerImport(
                     currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
               processConfigurationClass(candidate.asConfigClass(configClass), exclusionFilter);
            }
         }
      }
      catch (BeanDefinitionStoreException ex) {
         throw ex;
      }
      catch (Throwable ex) {
         throw new BeanDefinitionStoreException(
               "Failed to process import candidates for configuration class [" +
               configClass.getMetadata().getClassName() + "]", ex);
      }
      finally {
         this.importStack.pop();
      }
   }
}
```



# IOC扩展【Mybatis例】

## 需求

1. 将Mybatis的Mapper代理对象注册到IOC

## 难点、不恰点

Mapper代理对象个数不明，手动注册十分麻烦。即可排除以下方案：

1. 直接管理类的方法不可取 [无法生成指定的Mapper代理]，如@Component等方法
2. @Bean虽然可以注册指定的实例，但无法支持动态传参，即不能动态、批量的注册

## 思路转变

由于Mapper代理是由sqlSession获取的，而sqlSession又来自sqlSessionFactory，所以直接将sqlSessionFactory注册到IOC即可，但是目前仍无法传参。**更新需求：注册sqlSessionFactory**

**确定方法：** 实现一个FactoryBean，设置一个与mappeer代理对象关联的成员属性。

联系使用场景需要，1. 需要指定所有mapper的扫描路径【注解最佳】2. 当前应用的IOC容器【必须】 3. 为mapper代理生成唯一不重复的beanName，即需要beanName生成器【有更好，因为常规使用时只会用到一个代理对象，故可以通过类型获取】

## 方案确定

需要一个FactoryBean，最好有一个以当前应用IOC为入参的方法





















































