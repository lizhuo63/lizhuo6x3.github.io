# SpringMVC

## 接收含有属性值为json串的对象

```json
 # param为json串
{
"appId": "fd77450c-666a-4c9a-a6ff-88c13e98263a",
"platformChannelCode": "huizhifu_b2c",
"channelName": "汇支付B扫C",
"payChannel": "WX_MICROPAY",
"param":  "{ \"appID\": \"fd77450c-666a-4c9a-a6ff-88c13e98263a\", \"appSecret\": \"cec1a9185ad435abe1bced4b93f7ef2e\", \"key\": \"95fe355daca50f1ae82f0865c2ce87c8\", \"merchantId\": \"1556927311154872322\", \"payKey\": \"95fe355daca50f1ae82f0865c2ce87c8\" }",
"appPlatformChannelId": "1"
}
```

**GetMapping不能与@RequestBody同框出现**

**前端的Long长度大于17位时会出现精度丢失的问题**，为该字段添加注解`@JsonSerialize(using= ToStringSerializer.class)`，可将Long自动转为string类型



# UnKown

## component required a bean of type ‘XXX‘ that could not be found

+ 建议可靠Mybatis的maper配置(接口扫描，xml配置)



##  =======================

# 项目二

## No converter found capable of converting from type [org.bson.types.ObjectId] to type [java.lang.Long]

mongodb是文档型数据库，会以json字符串来接收和存储，返回的也当然是String型的数据，是无法直接转换从数值型的，了解到这个，就自行debug吧



## @Id

mongodb有默认的行_id，用来做唯一键，项目中的entity需要使用@Id来与这个注解进行映射，使被注解的字段成为主键字段。它可拦截getId，将被@Id注解到的属性值进行返回，通过setXxxId可实现为 _id赋值。

## 忽略匹配

在`Spring Data`中使用`MongoDB`时，插入数据会添加一个`_class`字段，这个字段是用来映射`POJO`的，使用`ExampleMatcher`，默认情况下会匹配所有字段，因此，如果实体类的包名改变了，`_class`字段就不会匹配，这样就无法正确地得到查询结果。把`_class`添加进`IgnorePath`。



## 属性""转null

```java
@Component
public class StringEditor extends PropertyEditorSupport {
   //setAsText完成字符串到具体对象类型的转换，
   @Override
   public void setAsText(String text) throws IllegalArgumentException {
      if (text == null || "".equals(text.trim())) {
         text = null;
      }
      setValue(text);
   }

   //getAsText完成具体对象类型到字符串的转换。
   @Override
   public String getAsText() {
      if (getValue() != null) {
         return getValue().toString();
      }
      return null;
   }
}
```



```java
@RestControllerAdvice
public class GlobalControllerAdviceController {
   @Autowired
   private StringEditor stringEditor;
   //WebDataBinder是用来绑定请求参数到指定的属性编辑器，可以继承WebBindingInitializer
   //来实现一个全部controller共享的dataBiner
   @InitBinder
   public void dataBind(WebDataBinder binder) {
      ///給指定类型注册类型转换器操作
      binder.registerCustomEditor(String.class, stringEditor);
   }
}
```



## 异常处理

1. 声明异常类
2. 定义异常枚举
3. 创建异常全局处理器
4. **将异常全局处理器加入扫描**



## mongodb gridFs.filex查询

19:22:55.959 [http-nio-31001-exec-2] ERROR c.l.f.exception.ExceptionCatch - catch exception:Value expected to be of type DOCUMENT is of unexpected type STRING