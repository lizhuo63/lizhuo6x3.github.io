# 反射

### 反射的作用与优缺点

反射机制能获取指定类的所有属性和方法；能在运行时动态地加载指定的类。

**优点**：能在运行时动态获取类的实例，提高了Java的灵活性；

**缺点**：反射时需要先找查找类资源，然后通过类加载器创建实例。过程繁琐，效率较低。解决办法。

+ 通过setAccessible(true)关闭JDK的安全检查来提升反射速度

### 反射的实现原理

[答案](/article/required_skills/java/java_reflex?id=反射的原理)

### 反射的4种方式

[答案](/article/required_skills/java/java_reflex?id=反射（获取class对象）的四种方式)

### 如何通过反射创建类的实例对象

1. `clazz.newInstance()`：通过Class对象调用newInstance()，但是要求该Class对象对应的类有无参构造器。

   ```java
   Class clazz=Class.forName("reflection.Person"); 
   Person p=(Person) clazz.newInstance();
   ```

2. `constructor.newInstance()`：通过Constructor 对象的调用newInstance()，

   ```java
   Class clazz=Class.forName("reflection.Person"); 
   Constructor c=clazz.getDeclaredConstructor(String.class,String.class,int.class);
   Person p1=(Person) c.newInstance("李四","男",20);
   ```

### 如何进行暴力反射

通过调用setAccessible(true)。

```java
//对A类中私有成员字段y进行暴力反射
Field fy=a.getClass().getDeclaredField("y");
fy.setAccessible(true);
System.out.println(fy.get(a));
```

### java支持对哪些类型进行反射

![image-20220817042952534](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220817042952534.png)



# 代理

### 什么是java代理，它的作用，它是怎么实现的

代理就是间接地通过代理对象去访问目标对象，而对于用户而言，他只能访问到代理对象。

**作用：**

1. 在不修改原代码的基础上，实现扩展和增强；
2. 起到了封装效果，隐藏了代码实现的过程和细节。
3. 代码解耦，在代理中可以通过参数判断来选择执行逻辑，灵活方便。

**原理：**

代理类在率先实现目标类的某个接口后，代理对象将会拦截对被代理对象的方法调用，在这个阶段，代理对象可以自由的对被代理对象的执行逻辑进行更，以此实现扩展和增强。

### 如何使用JDK代理

1. 创建代理对象
   1. 代理类实现目标类接口
   2. 创建InvocationHandler调用处理器
      1. 声明一个以目标类对象为入参的构造
      2. 实现invoke()
2. 代理对象代理调用目标类方法。proxy.hello();

### JDK代理类为何要与目标类实现相同的接口

1. 代理类对象创建后，需要通过直接调用目标类的方法来实现代理调用，就势必要求代理类与目标类之间相关联，而代理类通过字节码生成之后会默认继承Proxy类，无法继续继承目标类【单继承】，所以只好实现相同的接口。

   ```java
   public final class $Proxy0 extends Proxy implements HelloService {}
   ```

2. 此外，如果实现相同的接口，即上述代码没有`implements HelloService`，那么它通过字节码生成的就会是Proxy对象，且无法转换为上层接口，继而无法直接调用目标类的方法。

### 静态代理、jdk动态代理和cglib动态代理的区别

**静态代理：** 

**jdk动态代理：** 基于反射实现，通过让代理类实现相同的接口来实现对目标类的代理调用。

**cglib动态代理：** 基于ASM技术在生成代理类时继承目标类，无需接口。