# 什么是反射

Java属于先编译再运行的语言，程序中的类在编译期即可确定，而若在运行期间需要动态加载某些类时，这些类却没有被加载到JVM。所以需要在运行时动态地创建对象并调用其属性，这便是反射。它无关运行期的具体对象实例，本质是JVM通过class对象进行反编译，从而获取对象的各种信息。

![image-20220614214606742](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220614214606742.png)

## 实用反射

## 获取类对象

```java
Class.forName("全限定名")  
Class clazz = 类名.class
Class clazz = 对象.getClass();
```

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
//获取声明类及其父类（所有可访问）的public字段
Field[] fields = clazz.getFields();
//获取声明类的所有字段
Field[] declaredFields = clazz.getDeclaredFields(); 
//获取声明类及父类指定（public）字段 
Field name = clazz.getField("name"); //仅限public
//获取声明类（所有）指定字段 
Field name = clazz.getDeclaredField("name");  
```

## 获取方法

```java
//获取声明类及其父类（所有可访问）的public方法
  Method[] methods = clazz.getMethods();
//获取声明类的所有字段
  Method[] declaredMethods = clazz.getDeclaredMethods();
 //获取声明类及父类指定（public）字段【方法名，参数Class类】  
  Method method = clazz.getMethod("eat", String.class);
//获取声明类（所有）指定字段
  Method drink = clazz.getDeclaredMethod("drink", null);
```

