## 执行流程

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220220162249130.png)

1. 用户发送请求至前端控制器 DispatcherServlet。
2. DispatcherServlet 调用 HandlerMapping 处理器映射器获取执行器链。得到xxHandler之后再由处理器适配器HandlerAdapter去匹配具体的Handler实现。
3. 执行处理器方法，返回MV对象（完整model+逻辑view[仅含有名称]）
4. 调用视图解析器解析得到完整视图对象。
5. 有模板引擎将model填充到view中
6. 返回响应

### 使用过哪些springMVC的注解，都是干啥的

1. @Controller 负责处理DispatcherServlet分发的请求
2. 2.@RequestMapping 处理请求地址映射，作用于类和方法上
3. @Autowired 根据类型注入
4. @Qualifier 当一个接口存在多个实现类时，可以根据名称注入
5. @ResponseBody 用于异步请求返沪json数据
6. @PathVariable 在使用Resultful风格开发时，可使用该注解配上对应url中参数的名称来获取参数的值
7. @RequestParam 两个属性value、required，value用来指定参数名称，required用来确定参数是否必须传入。
8. @RequestHeader 可以把request请求header部分的值绑定到参数上
9. @CookieValue 把request请求header中的cookie绑定到参数上
10. @RequestBody 可指定请求数据格式application/json, application/xml
11. @ModelAttribute 为请求绑定需要从后台获取的值