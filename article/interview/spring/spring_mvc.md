## 执行流程

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220220162249130.png)

1. 用户发送请求至前端控制器 DispatcherServlet。
2. DispatcherServlet 调用 HandlerMapping 处理器映射器获取执行器链。得到xxHandler之后再由处理器适配器HandlerAdapter去匹配具体的Handler实现。
3. 执行处理器方法，返回MV对象（完整model+逻辑view[仅含有名称]）
4. 调用视图解析器解析得到完整视图对象。
5. 有模板引擎将model填充到view中
6. 返回响应
