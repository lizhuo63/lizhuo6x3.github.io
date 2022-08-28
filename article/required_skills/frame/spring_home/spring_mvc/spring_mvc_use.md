# 注解大全





# 入参属性绑定器

## 手段一 PropertyEditorSupport

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

