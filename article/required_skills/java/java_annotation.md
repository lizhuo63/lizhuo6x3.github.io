# 什么是注解

注解就是程序元素的元数据，一种可由编译器识别的附带信息，可看作是给编译器看的注释。

# 注解的作用

- 编写文档，通过注解生成javadoc文档。
- 编译检查，通过注解让编译器在编译期间进行检查验证。
- 编译时动态处理，编译时通过注解动态处理，例如动态生成代码。
- 运行时动态处理，运行时通过注解动态处理，例如使用反射**注入实例**。(xml同效)

# 完整注解的结构

<img src="https://gitee.com/lizhuo6x3/gallery_0/raw/master/img/202111100031319.png" alt="image-20211110003123050" style="zoom:80%;" />

**Annotation接口**

```java
public interface Annotation {
    //被标注的对象是否与本注解同属一个annotation类型
    boolean equals(Object obj);

    //返回此annotatio的哈希值
    int hashCode();

	//返回此annotation的字符串表示
    String toString();

    //返回此annotation的注释类型。
    Class<? extends Annotation> annotationType();
}
```

**java元注解**

+ @Target

  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Target {
     //指明该注解可标注的元素范围
      ElementType[] value();
  }
  ```

+ @Retention 

  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Retention {
   	//定义注解的保留策略
   	 //SOURCE：注解保留在源码中，编译器编译时将其丢弃
   	 //CLASS：注解保留在class文件中，运行时VM不保留，于类加载时丢弃【默认策略】
   	 //RUNTIME：注解保留在class文件中，运行时由VM保留注释，可反射读取
      RetentionPolicy value();
  }
  ```

+ @Documented

  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
      //将该注解的内容纳入用户API
      public @interface Documented {
  }
  ```

+ @Inherited

  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
      //指明该注解可被继承
      public @interface Inherited {
  }
  ```

+ @Repeatable

  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Repeatable {
     //标识该注解可以在同一个标注对象上使用多次。
      Class<? extends Annotation> value();
  }
  ```

  

**Java常见的自带注解**

- @Override - 检查该方法是否是重写方法。
- @Deprecated - 标记过时方法。如果使用该方法，会报编译警告。
- @SuppressWarnings - 指示编译器去忽略注解中声明的警告。
- @FunctionalInterface - 标识一个匿名函数或函数式接口。

# 注解是如何起作用的

编译时编译器会对注解符号进行处理并附加到class结构中的attributes属性中。JVM对于类、字段、方法，在class结构中都有自己特定的表结构，而且各自都有自己的属性，而对于注解，作用的范围可以在类上，也可以在字段或方法上，这时编译器会对应地将注解信息存放到类、字段、方法自己的属性上。

当JVM加载文件字节码时，就会将RuntimeVisibleAnnotations属性值(内容将以value=注解名的形式记录)保存到所在类的Class对象中，于是就可以通过Xxx.getAnnotation(Test.class)获取到Test注解对象，进而再通过Test注解对象获取到Test里面的属性值。

注解被编译后的本质就是一个继承Annotation接口的接口，即@Test就是“public interface Test extends Annotation”，当我们通过AnnotationTest.class.getAnnotation(Test.class)调用时，JDK会通过动态代理生成一个实现了Test接口的对象，并把将RuntimeVisibleAnnotations属性值设置进此对象中，此对象即为Test注解对象，通过它的value()方法就可以获取到注解值。



# 如何自定义注解

1. 声明注解 
2. 添加注解 
3. 实现注解处理器（传入操作对象）。
   1. 获取添加了注解的目标。
   2. 获取被注解对象
   3. 获取取注解对象
   4. 借助反射,然后根据注解及属性值做相应处理



## 声明注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface name {
    //定义value属性及默认值
   String value() default "tony";
}
```

## 添加注解&实现注解处理器

```java
public class AnnotationTest {
    //添加注解
   @namee
   String name;

   public static void main(String[] args) throws Exception {
      AnnotationTest annotationTest = new AnnotationTest();
      System.out.println(annotationTest.name);//null，此时注解还无法起作用
      init(annotationTest);
      System.out.println(annotationTest.name);//tony
   }
	//实现注解处理器，【即注解生效逻辑】
   public static void init(Object annotationTest) throws Exception {
      Class clazz = AnnotationTest.class;
      Field field = clazz.getDeclaredField("name");//获取被标注目标
      name annotation = field.getDeclaredAnnotation(namee.class);//获取注解对象
      field.set(annotationTest, annotation.value());
   }
}
```



**定义一个注解处理器的工具类让其自动赋值**

```java
public class AnnotationUtils {
   /**
    * @param clazz 被扫描的目标类
    * @param obj   被标注目标所在的对象
    * @throws Exception
    */
   public static void init(Class clazz, Object obj) throws Exception {
      //为字段注解赋值
      Field[] fields = clazz.getDeclaredFields();
      if (fields != null && fields.length > 0) {
         for (Field field : fields) {
            Annotation[] annotations = field.getDeclaredAnnotations();
            if (annotations != null && annotations.length > 0) {
               for (Annotation annotation : annotations) {
                  Method[] methods = annotation.annotationType().getDeclaredMethods();//获取当前注解的属性方法
                  if (methods != null && methods.length > 0) {
                     for (Method method : methods) {
                        field.set(obj, method.invoke(annotation, null));//调用该注解的属性方法赋值
                     }
                  }
               }
            }
         }
      }
   }
}
```