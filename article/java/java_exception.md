# 异常结构

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/2022-06-14_14-00.png)

程序错误分为三种：

+ 错误（Error）：是程序无法处理的错误，表示运行应用程序中较严重问题。
+ 运行时异常（Runtime Exception）：表示“JVM 常用操作”引发的错误，是不可查异常。
+ 编译异常：是必须进行处理的异常，否则程序就不能编译通过，是可查异常。必须要throws或者try catch。

# 异常处理

异常处理机制大体有：抛出异常，捕捉异常。

## throws声明异常

用于方法中可能有异常，但没有能力处理这种异常的情况。继而由上层的调用者去具体解决，上层调用者可直接解决异常或选择继续向上层抛出。

可查异常必须处理，否则编译不通过，常有try-catch捕获和throws上抛两种方案。

+ 父类方法没有抛，子类的覆盖方法就不能抛。(覆盖方法抛出的异常一定要比父类的范围小, 同样也不能抛出父类没有的异常，而且子类覆盖方法的访问权限不得低于父类的)。
+ 上层调用者抛的异常必须是下层的同类或父类异常。

```java
public class APP {
    public static int doa() throws ArithmeticException { //下层小异常
        int a = 1;
        int b = 0;
        return a / b;
    }
    public static void dob() throws Exception { //上层大异常
        int a = 1;
        a/=doa();
        System.out.println(a);
    }
    public static void main(String[] args) {
        try {
            dob();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
-- java.lang.ArithmeticException: / by zero
```



## throw抛出异常

throw总是出现在方法体中，用来抛出一个Throwable类型的异常。程序会在throw语句后立即终止，它后面的语句执行不到，然后在包含它的所有try块中（可能在上层调用函数中）**从里向外**寻找含有与其匹配的catch子句的try块。



## try catch

```java
// try无异常，不走catch
try {
    System.out.println(1);
    System.out.println(2);
} catch (Exception e) {
    System.out.println(3);
    System.out.println(4);
}finally {
    System.out.println(5);
    System.out.println(6);
}
------------1256-------------
//try有异常，走catch，遇到异常就跳出
try {
    System.out.println(1);
    int a = 1 / 0;
    System.out.println(2);
} catch (Exception e) {
    System.out.println(3);
    int a = 1 / 0;
    System.out.println(4);
}finally {
    System.out.println(5);
    int a = 1 / 0;
    System.out.println(6);
}
----------135---------------
```

# JVM的异常处理机制

如果一个方法中发生了异常，那么该方法就会创建一个异常对象并转交给 JVM（抛出异常）。可能要经过一系列的方法调用才进入到异常的抛出，该调用过程的有序列表叫做调用栈。

JVM 会顺着调用栈去查找看是否有可以处理异常的代码，如果有，就JVM就会将异常对象传过去并调用异常处理代码。如果 JVM 没有找到可以处理该异常的代码块，JVM 就会将该异常转交给默认的异常处理器（默认处理器为 JVM 的一部分），由默认异常处理器打印出异常信息并终止应用程序。
