__# 原理

## 执行流程

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220220162249130.png)

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220301203149593.png)

**细节:**

**1,2,3.**前端控制器**本身就是一个Servlet**，请求过来会进入它的doService方法下的doDispatch的getHandler【在HandlerMappings中遍历获取HandlerMapping】，返回处理器执行链HandlerExcutionChain，处理器执行链包含处理器xxHandler和处理器拦截器xxHandlerIntercepter。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_05-24-34.png)

**4.**从处理器执行链中取出xxHandler<u>(但是Handler有4种实现方式)</u>，再由处理器适配器HandlerAdapter去匹配具体的Handler实现。1.实现Controller接口 2.Controller注解 3.实现HttpRequestHandler 4.继承HttpServlet类。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_05-32-57.png)

**5,6,7.**执行HandlerAdapter的handler方法之前会**将handler转化为Controller**并执行handleRequest方法，此时返回的MV对象包含有完整的Model数据和逻辑视图(仅仅是试图名称)

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_05-42-49.png)

**8.9.**render渲染，让视图解析器ViewResolver根据视图名称获取完整的View视图对象

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_05-46-59.png)

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_05-50-32.png)

**10.**view通过模板引擎渲染model数据生成html并写入Response流中，然后转发出去。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_05-52-50.png)

**注意：**

​		处理器handler有多种实现方式所以他们会匹配不同的处理器映射器和适配器。1：simpleControllerxxx，2：requestMappingxxx，3：httpRequestxxx

## 参数封装

在执行指定handler方法之前，有参数解析器根据参数名获取参数

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_06-09-25.png)

## 核心组件

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_06-18-21.png)

### 拦截器

**拦截Handler**

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_06-19-53.png)

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_06-21-18.png)

**<u>遍历DispatchServlet中所有的拦截器，将与URL成功匹配的MappedInterceptor【封装了url和Interceptor】以及非MappedInterceptor【拦截所有】的拦截器封装到执行器链中。</u>**

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_06-23-00.png)

### 异常处理

拦截以下三个阶段中的异常

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_06-24-40.png)

## 容器关系

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_06-27-43.png)



# 过滤器&拦截器

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_08-13-35.png)

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_08-26-54.png" style="zoom:67%;" />

## **过滤器(Filter)：**

它是依赖于servlet容器的。它可以对几乎所有请求进行过滤，但一个过滤器实例只能在容器初始化时调用一次。它用于对传入的request、response进行提前过滤、设参，然后再传入servlet或者Controller进行业务逻辑操作。常用场景：设置字符编码（CharacterEncodingFilter）、XSSFilter(自定义过滤器)如：过滤低俗文字、危险字符等。

### 自定义过滤器

共有以下方案

+ @WebFilter 标注过滤器+@ServletComponentScan标注启动类，基于servlet，无法与spring联动
+ @Component + @Configuration【FilterRegistrationBean】，使用第三方bean注册，可实现与spring联动

```java
@ServletComponentScan
@SpringBootApplication
public class App { ... }

@WebFilter 
public class MyFilter implements Filter{
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("MyFilter 初始化了");
    }
    
-----------------------------------------------------------
@Component
public class MyFilter implements Filter{ ... }
    
@Configuration
public class FilterConfig {
   @Bean
    public FilterRegistrationBean filter() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new MyFilter());
        registration.addUrlPatterns("/*");
        registration.setName("filter");
        //设置优先级别
        registration.setOrder(1);
        return registration;
    }
```

## 拦截器(Interceptor)：

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_08-16-55.png)

它依赖于web框架，如SpringMVC。基于Java的反射机制实现的，是（AOP）的一种运用，同时一个拦截器实例在一个Controller生命周期之内可以多次调用，也可以对静态资源的请求进行拦截处理。

### 自定义拦截器

```java
@Component
public class MyInterceptor1 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        System.out.println("=1= 进入拦截器");
        return true; //放行
    }
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        System.out.println("=1= 执行完了");
    }
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        System.out.println("=1= 请求结束了");
    }
}

```

**配置**

**可通过order排序，默认顺序由上到下。**

```
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor1())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns("/LoginController/log");
        registry.addInterceptor(new MyInterceptor2())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/LoginController/log");
    }
}

```



## 区别

①拦截器是基于java的反射机制，而过滤器是基于函数回调。
　　②拦截器依赖与web框架，过滤器依赖与servlet容器。
　　③拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
　　④拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。
　　⑤在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。

⑥拦截器可以获取IOC容器中的各个bean，而过滤器就不行

拦截器是被包裹在过滤器之中的。



# 异常处理

## 使用@ExceptionHandler以及@ControllerAdvice

在Controller中添加@ExceptionHandler注解的方法来处理异常，这种方式只会对自身Controller的异常有效； 此外配合@ControllerAdvice注解来创建异常解析处理器，这种方式定义的异常处理方法将是全局的，对所有Controller抛出的异常都有效；

```java
// 处理所有@RestController注解标注的控制器
@ControllerAdvice(annotations = RestController.class)
// 处理包路径下的所有控制器
// @ControllerAdvice(basePackages = "org.example.controllers")
public class GlobalException {
    @ExceptionHandler(SQLException.class)
    public R mySqlException(SQLException e) {
        if (e instanceof SQLIntegrityConstraintViolationException) {
            return R.error("该操作涉及关联数据，操作失败！");
        } else if (e instanceof SQLTimeoutException) {
            return R.error("SQL超时！");
        } else if (e instanceof SQLException) {
            return R.error("数据库异常，操作失败!");
        } else {
            return R.error("服务器发生了错误！");
        }
    }
}
```



# 参数类型处理

+ **日期格式化：**@DateTimeFormat(pattern="yyyy-MM-dd hh:mm:ss")

+ **数值格式化：**@NumberFormat

  + **tyle：**Style.NUMBER(正常数字类型)，Style.CURRENCY(货币类型)，Style.PERCENT(百分数类型)
  + **pattern：**"#,###"。

+ **类型转器：**

  ```java
  //把字符串转换日期
  public class StringToDateConverter implements Converter<String,Date>{
      public Date convert(String source) {
          // 判断
          if(source == null){
              throw new RuntimeException("请您传入数据");
          }
          DateFormat df = new SimpleDateFormat("yyyy-MM-dd");
  
          try {
              // 把字符串转换日期
              return df.parse(source);
          } catch (Exception e) {
              throw new RuntimeException("数据类型转换出现错误");
          }
      }
  }
  ```



# 文件上传、下载

## 上传

```xml
<!-- 核心依赖 -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>

<!-- 配置文件解析器对象 -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="50000000" />
</bean>

<!--上传页面 -->
<form action="/upload" method="post" enctype="multipart/form-data">
        选择文件：<input type="file" name="upload" /><br/>
        <input type="submit" value="上传" />
</form>
```



```java
@RequestMapping("/upload")
    public String fileupload(HttpServletRequest request) throws Exception {
        System.out.println("文件上传...");
        // 使用fileupload组件完成文件上传
        // 上传的位置
        String path = request.getSession().getServletContext().getRealPath("/uploads/");
        // 判断，该路径是否存在
        File file = new File(path);
        if (!file.exists()) {
            // 创建该文件夹
            file.mkdirs();
        }
        // 解析request对象，获取上传文件项
        DiskFileItemFactory factory = new DiskFileItemFactory();
        ServletFileUpload upload = new ServletFileUpload(factory);
        // 解析request
        List<FileItem> items = upload.parseRequest(request);
        // 遍历
        for (FileItem item : items) {
            // 进行判断，当前item对象是否是上传文件项
            if (item.isFormField()) {
                // 说明普通表单向
            } else {
                // 说明上传文件项
                // 获取上传文件的名称
                String filename = item.getName();
                // 把文件的名称设置唯一值，uuid
                String uuid = UUID.randomUUID().toString().replace("-", "");
                filename = uuid + "_" + filename;
                // 完成文件上传
                item.write(new File(path, filename));
                // 删除临时文件
                item.delete();
            }
        }
        return "success";
    }
```

## 下载

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    <table>
        <tr>
            <td>被下载的文件名</td>
        </tr>
        <!--遍历 model中的 files-->
        <c:forEach items="${files}" var="filename">
            <tr>
                <td>
                    <a href="${pageContext.request.contextPath }/down?filename=${filename}">${filename}</a>
                </td>
            </tr>
        </c:forEach>
    </table>
</body>
</html>
```



```

```





# 注解开发

### 参数绑定

### javaBean

请求参数与**Javabean实体类**的绑定：类名必须与请求参数名称相同

```java
@RequestMapping("/login")
public String login(User user, Model model) {
```

请求参数与J**avabean属性值**的绑定：属性名称必须与请求参数名称相同

```java
@RequestMapping("/login")
public String login(String name, String pwd, Model model) {
```

### @RequestBody

将请求体中的参数封装为对象

### @RequestMapping

+  value， method；定义处理器的映射路径和方法类型

+  consumes，produces 定义处理器内容类型
   + consumes：指定接收的内容类型
   + produces：指定返回的内容类型【仅当request请求头中的(Accept)类型中包含该指定类型才返回】

+  params：必须包含某些参数值，才让该方法处理！
+  headers：必须包含某些指定的header值，才能该方法处理。

### @PathVariable

将请求URL上占位符的参数映射到指定形参上

```java
 @RequestMapping(value="/user/{userId}/roles/{roleId}",method = RequestMethod.GET) 
 public String getLogin(@PathVariable("userId") String userId, 
 @PathVariable("roleId") String roleId){ 
```

### @RequestParam

把请求中指定名称的参数给控制器中的形参赋值。使得任意指定参数可成功获取表单数据

### @RequestHeader

用于获取请求头参数。value：提供消息头名称；required：是否必须有此消息头

![image-20220220211917159](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220220211917159.png)

### @CookieValue

用于把指定 cookie 名称的值传入控制器方法参数。value：指定 cookie 的名称；required：是否必须有此 cookie

![image-20220220212902914](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220220212902914.png)

### @SessionAttributes

用于控制器方法间的参数共享。value：用于指定存入的属性名称；type：用于指定存入的数据类型

![image-20220220213435925](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220220213435925.png)

model.addAttributes(键值对)，可将键值对直接存入requestContext对象中

![image-20220220213531192](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220220213531192.png)

将存入requestContext对象中的键值对另存一份到sessioncontext对象中

## Json

![image-20220220184310621](https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/image-20220220184310621.png)



# 参数接收