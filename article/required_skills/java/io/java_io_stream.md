# IO流介绍

IO流主要解决的是数据的读入和写出，它涉及到数据的两端流向，而这两个端点在操作系统中以文件的形式体现，Java中通常以 File类【还有字节数组、字符数组】 展示。

## File类 (数据传输的端点)

```java
// 常用构造
File(String filePath);
File(String parentPath,String childPath);
File(File parentFile,String childPath);

//=========常用功能API============
getName();//获取文件名
getAbsolutePath();//获取绝对路径
getParentFile();//获取父级文件夹
length();//获取文件的长度
exists();
delete();//删除文件，不删目录
createNewFile();//创建文件
mkdirs();//创建多级目录
canRead();
canWrite();
isFile();//是文件
isDirectory();//是目录
listFiles();//获取子一级的所有File数组
```

# 流的理解

![image-20220705233032523](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220705233032523.png)

我们将A的数据传给B，需要A派出一个 **输入流** 从A中取出部分数据载入临时容器 (可大可小)，载入完毕后交给机器内存进行中转，随后B派出一个 **输入流** 从机器内存中取出数据并写入。不断循环直至临时容器无未读数据可装了。

## 流的分类

**输入流和输出流：**

**节点流和处理流：**

节点流：直接与数据源相连，在数据传输中必不可少

处理流：是装饰模式和适配器模式的体现。用于增强节点流，如提速、简化操作等，起到辅助和定制作用。

![image-20220705204407404](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220705204407404.png)

**字符流和字节流：**

字节流：以字节为最小单位传输，可以处理任意形式的数据

![image-20220705205552737](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220705205552737.png)

![image-20220705205745264](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220705205745264.png)

字符流：以字符为最小单位，仅支持文本数据的处理

![image-20220705210120369](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220705210120369.png)

![image-20220705221053367](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220705221053367.png)



# 流的操作

## 节点流：

### 文件流：

数据端点是文件，最常用。

**字节流：**

```java
fis;//FilsInputStream(file)
fos;//FilsOutputStream(file)
//准备数据的传输容量(容器)
byte[] buf=new byte[1024];
int len;
while((len=fis.read(buf))!=-1){//将数据载入传输容器
		fos.write(buf,o,len);//将读取的有效数据写出
}
fis.close();
fos.close();
```

**字符流：**

```java
fr;//FileReader(file)
fw;//FileWriter(file)
//准备数据的传输容量(容器)
Char[] buf=new Cahr[1024];
int len;
while((len=fr.read(buf))!=-1){//将数据载入传输容器
		fw.write(buf,o,len);//将读取的有效数据写出
}
fr.close();
fw.close();
```

### 非文件流

+ **ByteArrayInputStream、ByteArrayOutputStream：**数据源端点是字节数组
+ **CharArrayInputStream、CharArrayOutputStream：**数据源端点是字符数组
+ **PipedInputStream、PipedOutputStream：**用于线程之间传递信息，需要将两个流connect起来。
+ 

## 处理流：

Java的IO模块采用了装饰模式的设计方案，体现在通过缓冲流可以动态组装节点流，来达到定制用户所需要的流实例的目的。还有适配器模式，体现在转换流上。

### **缓冲流：**

为包装的节点流创建了一个默认的缓冲区8192 byte。机器内存直接从缓冲区【内存中】中获取数据。不再从硬盘中读数据。写数据时先将数据写道输出缓冲区，然后一次性刷入硬盘。

![image-20220706000647631](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220706000647631.png)

字节流：

```java
fis;//FilsInputStream(file)
fos;//FilsOutputStream(file)
BufferedOutputStream bos=new BufferedOutputStream(fos);
BufferedIntputStream bis=new BufferedIntputStream(fis);
//准备数据的传输容量(容器)
byte[] buf=new byte[1024];
int len;
while((len=bis.read(buf))!=-1){//将数据载入输入缓冲区
		bos.write(buf,o,len);//将输出缓冲区的数据写出
}
bis.close();
bos.close();
```

字符流：

```java
fr;//FileReader(file)
fw;//FileWriter(file)
BufferedReader br=new BufferedReader(fis);
BufferedWriter bw=new BufferedWriter(fos);
String str;
while((str=br.readLine()))!=null){//按行读取
		bw.write(str);//将读取的有效数据写出
    bw.newLine();//换行
}
br.close();
bw.close();
```

### **数据流：**

能够将基本类型的数据进行传输，而且可以保留对应的数据类型。

```java
dbis;//DataIntputStream(bis);
dbos;//DataOutputStream(bos);

dbos.writeInt(5);
dbos.writeDouble(5.26);
dbos.writeChar('a');
dbos.writeUTF("hello");
dbos.writeBoolean(true);
dbos.close;

//要求写读顺序一致
System.out.println(dbis.readInt());
System.out.println(dbis.readDouble());
System.out.println(dbis.readChar());
System.out.println(dbis.readUTF());
System.out.println(dbis.readBoolean());
dbis.close();
```

### **对象流：**

能够传输引用的对象。

```java
obis;//ObjectIntputStream(bis);
obos;//ObjectOutputStream(bos);

obos.writeInt(5);
obos.writeDouble(5.26);
obos.writeChar('a');
obos.writeUTF("hello");
obos.writeBoolean(true);
obos.writeObject(new Date());
obos.close;

//要求写读顺序一致，且实现序列化接口
System.out.println(obis.readInt());
System.out.println(obis.readDouble());
System.out.println(obis.readChar());
System.out.println(obis.readUTF());
System.out.println(obis.readBoolean());
Date date = (Date)obos.writeObject();
System.out.println(data);
obis.close();
```

### **打印流：**

只有输出流，没有输入流。

```
system.out;
PrintStream ps = new PrintStream();
InputStream is= system.in;//读取键盘录入
```

### **转换流：**

对字节流和字符流进行转换

```java
//INputStreamReader，将 InputStream 转换为 Reader
//OutputStreamReader，将 OutputStream 转换为 Writer

InputStream in = System.in;//读取键盘录入
BufferedReader reader = new BufferedReader(new InputStreamReader(in));//增强为读取文本
PrintWriter writer = new PrintWriter(new File("g:/a.txt"));//获取输出流
String str ;
while (!(str = reader.readLine()).equals("exit")) {//打印“exit”退出
    writer.println(str);//换行写入文件
}
reader.close();
writer.close();
```

















































