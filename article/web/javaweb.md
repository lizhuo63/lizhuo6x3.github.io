# web项目粗概

## 启动经过

1. 项目启动，加载依赖的jar包。
2. web容器（tomcat）先提供一个全局上下文ServletContext.
3. web容器去读取web.xml文件，并且运行ContextLoaderListener监听器，该监听器因为实现了ServletContextListener接口，所以当发现容器生成了一个ServletContext实例的时候，便会执行ServletContextListener接口的初始化方法，在该初始化方法中根据contextConfigLocation指定的位置去读取spring的主要配置文件，然后生成web应用上下文WebApplicationContext，并且将其作为一个属性注入到ServletContext中。
4. 初始化WebApplicationContext以后，启动了“业务层”的spring容器，并开始加载并初始化applicationContext配置文件中所扫描的类。
5. 然后就是初始化filter，最后初始化servlet。

所以说作为web项目，WebApplicationContext的生成必须要在web容器存在的情况下才能实现，因为他需要ServletContext，而ServletContext是web容器生成的。

![image-20220220143545980](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220220143545980.png)

## 自定义组件

### listener

#### springboot项目

1. 创建

   ```java
   @WebListener
   public class DoveInitListener implements ServletContextListener
   {
       @Override
       //用来通知监听器web应用初始化过程已经开始。在所有filter和servlet初始化前会通知所有实现了ServletContextListener的对监听器。
       public void contextInitialized(ServletContextEvent servletContextEvent) {
     
       }
   
       @Override
       //用来通知servletContext即将关闭。在通知所有ServletContextListener监听器servletContext销毁之前，所有servlet和filter都已被销毁。
       public void contextDestroyed(ServletContextEvent servletContextEvent) {
       
       }
   }
   
   ```

2. 载入

只需要在SpringBoot 启动类中加上`@ServletComponentScan`注解即可

在Spring Boot中可以监听多种事件，比如：

+ pplicationStartedEvent：spring boot启动监听类
+ ApplicationEnvironmentPreparedEvent：环境事先准备
+ ApplicationPreparedEvent：上下文context准备时触发
+ ApplicationReadyEvent：上下文已经准备完毕的时候触发
+ ApplicationFailedEvent：该事件为spring boot启动失败时的操作

### Filter

```java
@Component
@Order(1) //在定义多个filter下，声明执行顺序
public class TransactionFilter implements Filter {

    @Override
    public void doFilter(
    ServletRequest request,
    ServletResponse response,
    FilterChain chain) throws IOException, ServletException{
		//搞事情，
    }
    
    注意：如不声明URL则将拦截所有请求。以下，可定制
    @Bean
    public FilterRegistrationBean<UrlFilter> loggingFilter(){
        FilterRegistrationBean<UrlFilter> registrationBean
                = new FilterRegistrationBean<>();

        registrationBean.setFilter(new UrlFilter());
        registrationBean.addUrlPatterns("/users/*");

        return registrationBean;
    }
```

