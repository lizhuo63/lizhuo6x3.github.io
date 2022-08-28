



# JVM数据模型

## klass模型

一般jvm在加载class文件时，会在方法区创建instanceKlass，表示其元数据，包括常量池、字段、方法等。 **java类在JVM中的存在形式 ： java类->c++的类 klass** 

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-07_09-31-31.png" style="zoom: 67%;" />

+ InstanceKlass：表示普通java类在JVM中的实例。
+ InstanceMirrorKlass：表示java.lang.Class对象，在java中获取到的class对象，就是其实例，存储在堆区，也叫镜像类。
+ InstanceRefKlass：用于表示java.lang.ref.Reference类的子类。
+ InstanceClasLoaderKlass:表示类加载器，主要用于遍历类加载器加载的类。
+ TypeArrayKlass: 表示基本类型数组。
+ ObjArrayKlass：表示引用类型数组

## OOP模型

 **java对象在JVM中的存在形式 ： java对象 -> c++的对象** 

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-07_09-31-59.png" style="zoom:67%;" />

+ InstanceOopDesc：表示除数组外的其它对象。
+ MarkOopDesc：保存了java对象的一些信息，如哈希码、GC分代年龄、偏向锁标记、线程持有锁、偏向线程的ID、偏向时间戳。
+ TypeArrayOopDesc：基本类型数组对象。
+ ObjArrayOopDesc：引用类型数组对象。

### OOP模型内存结构

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-07_09-34-19.png" style="zoom:67%;" />

**指针压缩**：64bit才有这项技术；达到的效果 指针8B->4B【8B对齐】 **原理**：8的二进制数为1000；任何8的倍数的二进制数都有3个0，存储时将其去除【右移3位】，使用时再添上【左移3位】

### 对象头

对象头区域分为三部分，Mark Word，类型指针，数组长度。

+ **Mark Word**：标记字。主要用来表示对象的线程锁状态，存放对象hashCode、GC次数。32bit占用：4B； 64bit占用：8B

+ **类型指针**(Klass pointer):指向Class信息的指针，表示该对象是哪个Class的实例。 开启**指针压缩**【64bit的技术】：4B； 关闭指针压缩：8B

+ **数组长度**，当对象不是数组对象时，该区域不占空间。 默认占用4B【ffff】

+ **【注意】：关闭指针压缩下，数组对象在数组长度之后还会后一次对其填充以<u>保证对象头8字节对齐</u>**

    ```
    非数组对象
    	开启指针压缩:
    		MarkWord8B+类型指针4B+数组长度0B=12B【4B填充实例数据】
        关闭指针压缩:  
        	MarkWord8B+类型指针8B+数组长度0B=16B【不用填充】
        	
    数组对象
    	开启指针压缩:
    		MarkWord8B+类型指针4B+数组长度4B=16B【不用填充对象头】
    		，，，依情况填充实例数据
        关闭指针压缩:  
        	MarkWord8B+类型指针8B+数组长度4B=20B【4B填充对象头】
            ，，，依情况填充实例数据
     
    为什么数组对象要两次对齐填充
    	**效率**
    1.由于普通对象就只有一个实例数据，解决了一个就解决了所有，所以普通对象填充对象头的意义不大，还会浪费内存。所以普通对象的实例数据直接接到类型指针后面就可以了，最后一次填充一劳永逸。
    2.对于数组对象，可能会包含多个实例数据。如果不进行对象头填充，像普通对象一样接到数组长度后面，那么对于不同字节长度的数据类型就要进行不同的填充实现，会增加代码的复杂度。
    ```

### 实例数据

 类的非静态属性，生成的数据就是对象的实例数据

### 对象填充区域

当一个对象的大小不足8的整数倍的时候。会填充字节。 默认8字节对齐。 假如一个对象是30字节，会默认填充2个字节，达到8字节对齐。