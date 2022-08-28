# 什么是反射

Java属于先编译再运行的语言，程序中的类在编译期即可确定，而若在运行期间需要动态加载某些类时，这些类却没有被加载到JVM。所以需要在运行时动态地创建对象并调用其属性，这便是反射。它无关运行期的具体对象实例，本质是JVM通过class对象进行反编译，从而获取对象的各种信息。

![image-20220614214606742](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220614214606742.png)

![image-20220817030635752](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220817030635752.png)

源码在类加载时最终会以InstanceKlass的形式保留在jvm的方法区，它记录了源码的所有内容，其中_java_mirror映射的是该C++结构对应在JVM中的java类型【它同时**是InstanceKlass的访问入口**】，会以对应的Class类对象的形式存储在堆中。通过拿到Class类对象就能获取源码的字节码信息即InstanceKlass，进而获取各种信息数据，由此实现反射。

## 反射（获取Class对象）的四种方式

![image-20220817040817998](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220817040817998.png)

+ **1. Class.forName("全限定名")** 

  支持编译阶段的反射,可将一个还未编译过多源码文件加载进来，多用于框架的配置类。

+ **2. ClassLoader.loadClass()**

  通过类加载器将.class字节码文件加载到JVM，和1的效果差不多[都存在类加载过程]，实际上1是在编译拿到.class文件后调用了2

+ **3. 类名.class**

  直接获取Class对象，不用执行类加载，该方式多用在方法的入参。

+ **4. 对象.getClass()**

  与3一样，无需执行类加载，多用于已存在对象实例的场景。

<u>【注意】：同一个.class文件只会被加载一次，加载时执行的ClassLoader.loadClass()默认要执行 `checkPackageAccess()` 进行访问检查，可通过 `accessibleObject.setAccessible(true)` 关闭检查，实现优化。</u>

![image-20220817041953973](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220817041953973.png)

**如下元素【不限于】都具有对应的Class对象：**

![image-20220817042952534](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220817042952534.png)

## 获取实例

```java
clazz.newInstance();//获取类的实例对象
```

## 获取构造

```java
clazz.getDeclaredConstructors();　　　　    //获取所有的构造函数
clazz.getDeclaredConstructor(参数类型);　  //获取一个所有的构造函数
clazz.getConstructors();　　　　　　　　　//获取所有公开的构造函数
clazz.getConstructor(参数类型);　　　　　 //获取单个公开的构造函数
```

## 获取字段

```java
Field[] fields = clazz.getFields(); //获取声明类及其父类（所有可访问）的public字段
Field[] declaredFields = clazz.getDeclaredFields();  //获取声明类的所有字段
Field name = clazz.getField("name"); //获取声明类及父类指定（仅限public）字段 
Field name = clazz.getDeclaredField("name");  //获取声明类（所有）指定字段 
```

## 获取方法

```java
Method[] methods = clazz.getMethods(); //获取声明类及其父类（所有可访问）的public方法
Method[] declaredMethods = clazz.getDeclaredMethods(); //获取声明类的所有字段
Method method = clazz.getMethod("eat", String.class);  //获取声明类及父类指定（public）字段【方法名，参数Class类】  
Method drink = clazz.getDeclaredMethod("drink", null); //获取声明类（所有）指定字段
```

































