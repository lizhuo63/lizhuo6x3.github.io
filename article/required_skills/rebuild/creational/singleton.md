# 背景

​		某些特殊情况下或者特定有需求，要让目标只允许创建一个实例。

**核心：**

1. 私有化构造方法，阻断外界随意创建的可能
2. 构造一个私有的静态实例引用
3. 提供以供公开的访问方法



# 静态类

​		静态类的⽅式可以在第一次运行的时候直接初始化Map类，同时这里我们也不需要延迟加载使⽤。多用于对外提供全局访问，类似池。

```
public class Singleton_00 {
		public static Map<String,String> cache = new ConcurrentHashMap<String,String>();
}
```



# 内部类

​		相当于将实例创建的方式，用一个静态内部类封了起来。

```java
public class Singleton_01 {
		private static class SingletonHolder {
				private static Singleton_01 instance = new Singleton_01();
		}
		private Singleton_01() {
		}
		public static Singleton_01 getInstance() {
				return SingletonHolder.instance;
		}
}
```



#  饿汉模式(线程安全)

​		与静态类基本一致。

```java
public class Singleton_02 {
		private static Singleton_02 instance = new Singleton_02();
		private Singleton_02() {
		}
		public static Singleton_02 getInstance() {
		return instance;
		}
}
```



# 懒汉模式(线程不安全)+同步代码块

​		对于用到时才创建这一动作本来就是非线程安全的，可以想象在极端的多线程并行情况下可不就是会创建多个实例嘛。可为getInstance( )方法加上一个同步锁，来达到线程安全的目的。

```java
public class Singleton_03 {
		private static Singleton_03 instance;
		private Singleton_03() {
		}
		public static synchronized Singleton_03 getInstance(){
		if (null != instance) return instance;
		instance = new Singleton_03();
		return instance;
		}
}
```



# 懒汉模式(线程不安全)+双重锁校验

​		对同步代码块的优化

```java
/**
 * 双重检验：改善了懒汉式的非线程安全问题
 * 特点：避开了同步方法，提高了程序效率
 * 注意：极端情况下仍有问题===》
 *      CPU存储系统：内存 -> 寄存器 -> 缓存 -> 内存 ；程序原数据被载入内存后，cpu为效率考量会将其通过寄存器载入缓存，载入寄存器的数据在本行指令结束之后会将其释放移入缓存，缓存的数据仅本线程可访问
 *      1.若E-instance实际值（内存）为null -> 寄存器【null，执行1.if (INSTANCE == null)后释放】 -> 缓存 null
 *      2.E-instance缓存值为null -> 通过了验证，执行实例化指令
 *   解决方法：volatile关键字：避开缓存，直接从内存中读取，更新数据
 */
public class DoubleCheckSingleton {
    private DoubleCheckSingleton() {
    }
    public static volatile DoubleCheckSingleton INSTANCE;
    public static DoubleCheckSingleton getInstance() {
        //“若E-instance为空会将null存入缓存，极端情况：本线程再次步入此方法，会从缓存中获取null，从而再次执行实例化”
        if (INSTANCE == null) {//第一次判断：若已有实例，直接返回；设A,B步入
            synchronized (DoubleCheckSingleton.class) {//同步构造方法，防止多线程的多创建，设A获取锁，B等待
                if (INSTANCE == null) {//第二次判断：对首次步入的线程不作处理，当A实例化后，B步入，若无此判断，B将直接执行new指令，破坏单例
                    //“E-instance完成实例化更新了数据INSTANCE=DoubleCheckSingleton@2b46d869并将其存入内存，更改了实际值，他线程会获取非空的实际值步入判断，”
                    INSTANCE = new DoubleCheckSingleton();
                }
            }
        }
        return INSTANCE;
    }
}
```































