# java

## 基础杂篇

### switch

**语句能否作用在byte上，能否作用在long上，能否作用在string上？**

+ 1.7之前，在switch(expr1)中，expr1只能是一个整数表达式或者枚举常量，
  整数表达式可以是int基本类型或Integer包装类型。byte,short,char都可以隐式转换为int。
+ 不过，在1.7版本之后switch就可以作用在string上了。

由于，所以，这些类型以及这些类型的包装类型也是可以的。

long和String类型都不符合switch的语法规定，并且不能被隐式转换成int类型，
所以，它们不能作用于swtich语句中。

### int与Integer

+ interger是对象，两个new的integer不会相等
+ 非new 生成的对象均来自常量池，数值相等的情况下equals为true【-128~127之外的仍为false，缓存】
+ int 5与integer 5 的equals为true，数值比较时会拆箱

### String  StringBuffer StringBuilder

+ 都是final类不允许被继承

1. **String不可变**。为了提高使用效率，实现了具体字符串的唯一性(多个相同字符串引用指向同一个对象)，并且避免多次计算hashcode，需要保证String不可变。另外String多用于URL访问，为保证安全，也需要String不可变。
   + 使用"+"连接时，会调用StringBuilder.append()
2. StringBuilder线程不安全
3. StringBuffer 线程安全

### Comparable和Comparator

+ **Comparable 接口：**多用于自然排序，且只能基于一个字段进行排序，不能根据需要选择对象字段来对对象进行排序【排序目标是固定的】。自定义类需要实现 Java提供 Comparable 接口的 compareTo(TOBJ)方法。它被排序方法所使用，应该重写这个方
  法，如果“this”对象比传递的对象参数更小、相等或更大时，它返回一个负整数、0 或正整
  数。它
+ **Comparator 接口：**可以提供不同的排序算法实现两个对象的特定字段的比较（比如，比较员工这个对象的年龄），该接口的 compare(Objecto1, Object o2)方法的实现需要传递两个对象参数，若第一个参数小于、等于、大于第二个参数，返回负整数、0、正整数



### 内部类

<u>【问题】外部类变量是怎么传递给内部类的</u> ：通过构造器（this）。内部类【包括局部~、匿名~】持有外部类的引用。

内部类在形式上就是外部类的一个属性而已，故而可分为成员内部类和静态内部类两种。

**成员内部类：** 它依赖于外部类，需要**通过`outClass.new InnerCLass();`来创建内部类**，而且它会保留外部类的this引用，所以它**可以直接访问外部类的任何字段【包括 normal、static、final、static final】**。此外，它**无法声明static方法和字段**，因为static直属于类，但成员内部类由依赖外部类实例来创建，会矛盾。但却**可以声明 static final 的字段**，因为final static的变量是存放在常量池中的，不涉及到类的加载。

**静态内部类：** 它不依赖于外部类，可直接创建，不能访问外部类任何非static的字段和方法。但可以声明任何类型的字段【包括 normal、static、final、static final】以及静态方法。

**单例应用：**

```java
 public class SingleTon{
  private SingleTon(){}
   
  private static class SingleTonHoler{
     private static SingleTon INSTANCE = new SingleTon();
 }
 
  public static SingleTon getInstance(){
    return SingleTonHoler.INSTANCE;
  }
```

静态内部类的优点是：外部类加载时并不需要立即加载内部类，内部类不被加载则不去初始化INSTANCE，故而不占内存【懒汉式】。即当SingleTon第一次被加载时，并不需要去加载SingleTonHoler，只有当getInstance()方法第一次被调用时，才会去初始化INSTANCE，第一次调用getInstance()方法会导致虚拟机加载SingleTonHoler类，这种方法不仅能确保线程安全，也能保证单例的唯一性，同时也延迟了单例的实例化。

**实现多继承：**

```java
	class Father {
		private String father = "father";
		public String getFather() { return father; }
	}

	class Mother {
		private String mather = "mather";
		public String getMather() { return mather; }
	}

	class Son {
		class Father_1 extends Father {}

		class Mother_1 extends Mother {}

		public void show() {
			System.out.println(new Father_1().getFather() + " " + new Mother_1().getMather());
		}
	}
```

### 局部内部类

局部内部类是定义在一个方法或者一个作用域里面的类，它的访问仅限于该作用域内。局部内部类就像是方法里面的一个局部变量一样，是不能有public、protected、private以及static修饰符的。

### 匿名内部类

它若要访问外部类的局部变量必须由 final 修饰，java8的Effectively final新特性会默认加上，自此，在匿名内部类中将无法对该变量进行更改【final】。

Lambda表达式并不是匿名内部类的语法糖，它是基于invokedynamic指令，在运行时使用ASM生成类文件来实现的。



## OOP

**继承：**

+ Java 中 static 方法不能被覆盖，因为方法覆盖是基于运行时动态绑定的，而 static 方法
  是编译时静态绑定的。
+ java 中也不可以覆盖 private 的方法，因为 private 修饰的变量和方法只能在当前类中使用，如果是其他的类继承当前类是不能访问到 private 变量或方法的，当然也不能覆盖。

**多态：**



## 反射

+ 获取Class对象的方法：Class.forName()、class.getClass()、ClassLoader.loadClass()。

1. Class.forName() 会对类解析初始化，而ClassLoader.loadClass()会跳过初始化，直接装载，链接。
2. Class.forName()在加载类时会执行静态代码块，而ClassLoader.loadClass()只有在调用newInstance()时才会执行静态代码块。
3. Class.forName()会使用调用者的类加载器来加载目标类，而ClassLoader.loadClass()需要指定类加载器。



## 执行与加载

构造器先与静态代码块执行，静态代码块只执行一次

### 数值运算

+ float x = 1；float x = 1.0f。无错：低精度可向高精度自行转换
+ float x = 1.0；。**出错：**浮点数默认是double的，高转低-->失败。
+ `-5+1/4+2*-3+5.0 = double -6.0`。当多个精度的数字同时进行运算时，最终结果以最高精度为准。



### 字符型常量和字符串常量的区别

+ 形式上: 字符常量是单引号引起的⼀个字符; 字符串常量是双引号引起的若⼲个字符
+ 含义上: 字符常量相当于⼀个整型值( ASCII 值),可以参与运算; 字符串常量代表⼀个地
  址值
+ 内存占用: 字符常量只占 2 个字节; 字符串常量占若⼲个字节

| 基本类型 | 位数 | 字节 | 默认值  |
| :------: | :--: | :--: | :-----: |
|   int    |  32  |  4   |    0    |
|  short   |  16  |  2   |    0    |
|   long   |  64  |  8   |   0L    |
|   byte   |  8   |  1   |    0    |
|   char   |  16  |  2   | 'u0000' |
|  float   |  32  |  4   |   0f    |
|  double  |  64  |  8   |   0d    |
| boolean  |  1   |      |  false  |

### 字符串创建

+ 字面量形式：`String str1="aaa"; String str2="aaa";`
  1. 先去堆中的字符串常量池中查找是否存在"aaa"这个字符串对象
  2. 若无就在字符串池中创建并返回引用，若有就直接返回对象地址
  3. str1==str2为true。
+ new( )形式：`String str1=new String("aaa"); String str2=new String("aaa");`
  1. 先在字符串常量池中查找有没有"aaa"这个字符串对象
  2. 若无，则先在字符串池中创建一个再在堆中创建一个，然后返回堆中的对象地址；若有，就直接在堆中创建，返回堆中的对象地址。
  3. str1==str2为false。

### 重载和重写的区别

+ **重载：**在编译时即可实现，在同一个类中体现，在同方法名下有不同的参数列表则视为重载，返回值类型和访问修饰符也可以不相同。
+ **重写：**在运行时才可实现，体现在子父类的继承中，要求方法名，参数列表，返回类型都相同。但子类函数的访问修饰权限不能低于父类，抛出的异常范围要⼩于等于⽗类，构造⽅法⽆法被重写。

### 封装 继承 多态

+ **封装：**把对象的属性私有化，同时提供能被外界访问属性的⽅法
+ **继承：**在已存在的类的基础上建⽴新类的技术，新类可以增加新的字段或方法
  + ⼦类拥有⽗类对象所有的属性和⽅法（包括私有属性和私有⽅法），但是⽗类中的私有属性
    和⽅法⼦类是⽆法访问，只是拥有而已。
+ **多态：**上级引用（继承重写、接口覆盖）执行下级实例，在运行期确定

### 接⼝和抽象类的区别

+ **关键字：**接口使用关键字interface来定义，使用implements定义其具体实现；抽象类使用关键字 abstract 来定义，使用 extends 关键字实现继承。
+ **子扩展：**接口可以多实现，但抽象类只能单继承。
+ **访问控制符：**接口中属性和方法的访问控制符只能是public，属性默认是public static final修饰的；抽象类中的访问控制符无限制，但抽象方法不能使用 private 修饰。
+ **方法实现：**接口中的static 和 default 方法必须有实现；抽象类中普通方法可以有实现，抽象方法不能有实现。
+ **静态代码块：**接口中不能使用静态代码块；抽象类中可以使用静态代码块
+ **设计层⾯：**抽象类是对类的抽象，是⼀种模板设计，⽽接⼝是对⾏为的抽象，是⼀种⾏
  为的规范。

### 成员变量与局部变量的区别

+ **形式上：**员变量可以被public、private、static等修饰符所修饰，但是局部变量不能被访问控制修饰符以及static所修饰。局部变量和成员变量都能被final修饰
+ **存储：**成员变量存在于堆内存，局部变量存在于栈内存。

+ **生命周期：**成员变量随着对象的创建而存在；局部变量随着方法的调用而自动消失。
+ **初始化：**成员变量会自动以类型的默认值而赋初值，但是局部变量则不会自动赋值。被final修饰的成员变量必须显式的赋值。

###  == 与 equals

+ **== :** 它的作⽤是判断两个对象的地址值是不是相等。即判断两个对象是不是同⼀个对象
+ **equals() :** 它的作⽤也是判断两个对象是否相等。但它⼀般有两种使⽤情况：
  + 类没有覆盖 equals() ⽅法：等价于“==”。
  + 类覆盖了 equals() ⽅法：用于⽐较两个对象的内容是否一致

### hashCode 与 equals

hashCode() 的作⽤是获取哈希值，用于确定元素在哈希表中的索引位置。在有散列表参与的集合中，会依次通过hashCode()和equals()来确定元素的最终位置，并且合理的hashCode()实现可以大大提高执行效率。

若两对象相同，则它们必定会覆盖在同一个桶中的同一个位置，即hashCode值相等且equals()返回true，相反，仅凭hashCode值相等是无法精确定位元素的最终位置的。如此一来，可知，在散列表支持的集合中，判断两元素相等是需要hashCode()和equals()共同协作的，必须同时重写才能起作用。

###  如何判断⼀个常量是废弃常量?

运⾏时常量池主要回收的是废弃的常量。假如在常量池中存在字符串 "abc"，若当前没有任何String对象引⽤该字符串常量的话，就说明常量 "abc" 就是废弃常量，如果这时发⽣内存回收的话⽽且有必要的话，"abc" 就会被系统清理出常量池。

### 如何判断⼀个类是⽆⽤的类?

需要同时满⾜下⾯ 3 个条件：

+ 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。
+ 加载该类的 ClassLoader 已经被回收。
+ 该类对应的 java.lang.Class 对象没有被引⽤，⽆法通过反射访问该类的⽅法。

###  为什么 Java 中只有值传递

按值调用表示方法接收的是调用者提供的值，而按引用调用表示方法接收的是调用者提供的变量地址。一个方法可以修改传递引用所对应的变量值，而不能修改传递值调用所对应的变量值。 Java总是采用按值调用。也就是说，方法得到的是所有参数值的一个拷贝，也就是说，方法不能修改传递给它的任何参数变量的内容。

### 深拷贝 vs 浅拷贝

**浅拷贝：**对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，只拷贝了当前层。

**深拷贝：**对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，相当于递归进行浅拷贝。



## 集合

### HashMap 的⻓度为什么是2的幂次⽅

HashMap在定位桶索引时，使用了位运算进行了优化【hash%length==hash&(length-1)】，但该式子成立的前提就是 **length是2的n次⽅**。

### 快速失败(fail-fast)和安全失败(fail-safe)

+ **fail-fast：**在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改就会抛出 Concurrent Modification Exception。
+ **fail-safe：**采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原
  有集合内容，在拷贝的集合上进行遍历。

## IO

### BIO,NIO,AIO 有什么区别?

+ **BIO (Blocking I/O):** 同步阻塞 I/O 模式，数据的读取写入必须阻塞在一个线程内等待其完成。在连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。
+ **NIO (Non-blocking/New I/O):** NIO 是一种同步非阻塞的 I/O 模型，它支持面向缓冲，基于通道I/O 操作方法。 NIO 提供了与传统BIO 模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。阻塞模式与BIO基本一致，但性能和可靠性都不好；非阻塞模式正好与之相反。
+ **AIO (Asynchronous I/O):** AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步
  非阻塞的 IO 模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不
  会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

# JVM

## 类加载

1. .java源代码→javac编译为字节码文件.class，并存放在硬盘上
2. 将.class文件加载到jvm的方法区内
   1. 找到字节码文件并载入**内存**
   2. 对字节码文件进行解析，将生成的运行时数据【即instanceKlass实例(class结构体)】存于**方法区**，并释放.class文件
   3. 在**堆区**生成该类的Class对象【**此时为空对象**】
3. 验证，检查该.class是否符合JVM规范
4. 准备
   1. 为静态变量【归属于class】分配内存、赋初值(若被final修饰就直接完成赋值)，并将其存于堆区。
5. 解析，有常量池中的符号引用定位到直接引用。
   + 继承的类，实现的接口 ——> 进行加载并存放于方法区
   + 字段 ——> 存放于堆
   + 类方法，接口方法 ——> 存放于方法区
6. 初始化
   1. 执行静态代码块clinit，完成静态变量的赋值。

# SpringMVC

## 执行流程

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220220162249130.png)

1. 用户发送请求至前端控制器 DispatcherServlet。
2. DispatcherServlet 调用 HandlerMapping 处理器映射器获取执行器链。得到xxHandler之后再由处理器适配器HandlerAdapter去匹配具体的Handler实现。
3. 执行处理器方法，返回MV对象（完整model+逻辑view[仅含有名称]） 
4. 调用视图解析器解析得到完整视图对象。
5. 有模板引擎将model填充到view中
6. 返回响应

## SpringMVC加载流程





# Spring



## Bean作用域

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-13_22-18-44.png)

##  Bean生命周期

1. 实例化Bean
2. 依赖注入
3. 检查该Bean实现的相关Aware接口，并设置对应的资源
4. 检查该Bean是否自定义 init-method，有就调用
5. 检查该Bean是否实现了BeanPostProcessor 接口，将会调用postProcessAfterIn
   itialization(Object obj, String s)方法
6. Bean初始化成功，投入使用
7. 检查该Bean是否实现了DisposableBean这个接口，有就调用destroy()方法
8. 最后检查这个Bean是否自定义了destroy-method属性，有就调用自配置的销毁方法

## IOC

IOC 即“控制反转”，是一种设计思想，它利用java 反射机制。以前要由程序主动去创建依赖对象，耦合性很强，IOC 则是由容器来帮忙创建及注入依赖对象，对象只是被动的接受依赖对象。

DI即“依赖注入”**：**组件之间的依赖关系由容器在运行期决定的，即由容器动态的将依赖关系注入到组件之中。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

## AOP

意为面向切面编程，是OOP的延续，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。可用于日志记录、效率检查、事务控制。

## 循环依赖

- ①构造器的循环依赖：这种依赖spring是处理不了的，直接抛出BeanCurrentlylnCreationException异常。
- ②单例模式下的setter循环依赖：通过“三级缓存”处理循环依赖，能处理。
- ③非单例循环依赖：无法处理。原型(Prototype)的场景是不支持循环依赖的，通常会走到`AbstractBeanFactory类中下面的判断，抛出异常。`

**三级缓存：**

+ singletonObjects （一级缓存）俗称“单例池”“容器”，用于缓存创建完成的单例Bean
+ earlySingletonObjects（二级缓存）映射Bean的早期引用(不是完整的)，只是一个Instance.
+ singletonFactories（三级缓存） 映射创建Bean的原始工厂

当 Spring 为某个 Bean 填充属性时，首先会寻找需要注入对象的名称，然后依次执行 getSingleton() 方法得到所需注入的对象，而获取对象的过程就是依次从一级缓存、二级缓存中获、三级缓存中获取，如果三级缓存中也没有，那么就会去执行 doCreateBean() 方法创建这个 Bean。



# Mybatis

##  Dao 接⼝的⼯作原理

Mapper 接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Mapper接口生成代理对象 MappedProxy，代理对象会拦截接口方法，根据类的全限定名+方法名，唯一定位到一个MapperStatement并调用执行器执行所代表的sql，然后将sql执行结果返回。 Mapper接口里的方法，是不能重载的，因为是使用全限名+方法名 的保存和寻找策略。   



# Redis

## 使用场景

