# 什么是代理

就是代理方获取到了甲方的资源，并使用甲方的资源实现自己业务的过程，那么就很清楚，实现代理至少需要两样东西，即代理方和甲方资源。对应java层，就是代理类和目标类方法。

首先不考虑代码，单单捋一下实现手段和逻辑。

1. 引用目标类的方法，并在引用代码的前后附加代理逻辑。**实现难点**
   1. 如何灵活的确定代理的目标类  >>> 要获取到目标类的方法列表 >>> 要使用反射、
2. 拷贝目标类方法生成一个新的代理类，完全走代理类的逻辑。**实现难点**



# JDK动态代理的实现

1. 创建代理对象
   1. 代理类实现目标类接口
   2. 创建InvocationHandler调用处理器
      1. 声明一个以目标类对象为入参的构造
      2. 实现invoke()
2. 代理对象代理调用目标类方法。

```java
    public interface HelloService {
        void sayHello();
    }
    /******************************************************/
    public class HelloServiceImpl implements HelloService {
      @Override
      public void sayHello() {
        System.out.println("hello");
      }
    }
    /****************************************************/
    public class HelloInvocationHandle implements InvocationHandler {
        private Object target;
        public HelloInvocationHandle(Object target) {
            this.target = target;
        }
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("=======proxy before=======");
            Object result = method.invoke(target, args);
            System.out.println("=======proxy end=======");
            return result;
        }
    }
		/****************************************************/
public class HelloTest public static void main(String[] args) { 
    HelloService hello = new HelloServiceImpl(); 
    HelloInvocationHandle handle = new HelloInvocationHandle (hello); 
    HelloService proxy = (HelloService) Proxy.newProxyInstance( 		                                    Thread.currentThread().getContextClassLoader (), 
           hello.getClass().getInterfaces (),
           handle);
   proxy.sayHello();
}
```



**反编译得到的代理类源代码：**

```java
public final class $Proxy0 extends Proxy implements HelloService {
    private static Method m1; //equals()方法
    private static Method m3; // =========sayHello方法()
    private static Method m2; //toString()方法
    private static Method m0; //hashCode()方法
 
    //这里就是我们之前提交的InvocationHandler这个构造方法！！
    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }
    public final boolean equals(Object var1) throws  {
        try {
            return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    
    //代理类的sayHello()方法
    public final void sayHello() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    public final int hashCode() throws  {
        try {
            return ((Integer)super.h.invoke(this, m0, (Object[])null)).intValue();
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
            m3 = Class.forName("proxy.service.HelloService").getMethod("sayHello", new Class[0]);
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```



# cglib动态代理

```java
public class CglibTest1 {
	public static void main(String[] args) {
		//声明被代理对象 ,可有可无
		Service service = new Service();
		//创建增强器
		Enhancer enhancer = new Enhancer();
		//声明代理类
		enhancer.setSuperclass(Service.class);
		enhancer.setCallback(new MethodInterceptor() {
			/**
			 *
			 * @param obj 代理生成类的实例对象
			 * @param method 拦截方法
			 * @param args 方法入参
			 * @param proxy
			 * @return
			 * @throws Throwable
			 */
			@Override
			public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
				if (method.getName().equals("doServe")) {
					System.out.println("欢迎光临");
//					method.invoke(service, args); //1. 调用原对象的指定方法
//					proxy.invoke(obj, args);//2. 调用代理生成类实例的重写逻辑，匹配成功，会执行intercept(）中的逻辑，造成栈溢出
					proxy.invokeSuper(obj, args);//3. 调用代理生成类实例的父类逻辑，即 Service 的原有逻辑，效果等同于 1
					System.out.println("谢谢惠顾");
					return null;
				}
				return method.invoke(service, args);
			}
		});
		//构建代理生成类的实例对象
		Service proxy = (Service) enhancer.create();
		proxy.doServe("理发服务");
	}
```

```java
//main方法设置如下，可查看cglib的生成类
String sourcePath = "H:\\Study\\java8\\target\\classes\\cg";
System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, sourcePath);
```

原理，根据声明的代理类或者代理的接口，来重写它的方法，所有方法的实现均有两套

```java
    private static final Method CGLIB$doServe$0$Method;
    private static final MethodProxy CGLIB$doServe$0$Proxy;
    private static final Object[] CGLIB$emptyArgs;
    private static final Method CGLIB$doneServe$1$Method;
    private static final MethodProxy CGLIB$doneServe$1$Proxy;
    private static final Method CGLIB$equals$2$Method;
    private static final MethodProxy CGLIB$equals$2$Proxy;
    private static final Method CGLIB$toString$3$Method;
    private static final MethodProxy CGLIB$toString$3$Proxy;
    private static final Method CGLIB$hashCode$4$Method;
    private static final MethodProxy CGLIB$hashCode$4$Proxy;
    private static final Method CGLIB$clone$5$Method;
    private static final MethodProxy CGLIB$clone$5$Proxy;
```

以上 **Method** 是jdk的类，而 **MethodProxy** 是cglib的类，

```java
// 直接走代理目标【父类】的逻辑
final void CGLIB$doServe$0(String var1) {
        super.doServe(var1);
    }
//走自定义的增强逻辑
    public final void doServe(String var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
          	//走回调的intercept()
            var10000.intercept(this, CGLIB$doServe$0$Method, new Object[]{var1}, CGLIB$doServe$0$Proxy);
        } else {
            super.doServe(var1);
        }
    }
//=========包括其他无关紧要的方法，也有两套，都是一致的逻辑=============
    public final boolean equals(Object var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            Object var2 = var10000.intercept(this, CGLIB$equals$2$Method, new Object[]{var1}, CGLIB$equals$2$Proxy);
            return var2 == null ? false : (Boolean)var2;
        } else {
            return super.equals(var1);
        }
    }
```

