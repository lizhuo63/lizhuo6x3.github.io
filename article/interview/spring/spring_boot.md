# SpringBoot

### 自动装配原理

@SpringBootApplication里有三个重要注解：

+ @SpringBootConfiguration：声明定义Bean

+ @ComponentScan：扫描主配置类包的所有包
+ @EnableAutoConfiguration：**开启自动装配类**,而@EnableAutoConfiguration里也有两个重要注解。
  + **@AutoConfigurationPackage**：自动配置包，给Spring容器中导入一个Registrar注册器组件，它主要是**扫描第三方依赖的注解**如@MapperScan
  + **@Import(AutoConfigurationImportSelector.class)**——**核心注解**，导入第三方提供的bean的配置类。该类中有selectImports()方法，扫描所有jar包类路径下的META-INF/spring.factories文件，将扫描到的这些文件包装成properties对象，从properties中获取到EnableAutoConfiguration.class类名对应的值，将这些值添加到容器中，用这些类做自动配置功能。当然，它会根据配置类的条件判断配置类是否生效，若生效，则添加各种组件。

