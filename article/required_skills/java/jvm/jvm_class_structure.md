

# .class文件的结构

## class文件

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-08_08-26-34.png)

## 常量池

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-08_09-15-21.png" style="zoom:67%;" />

<u>info一共有17项内容门类</u>

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-08_09-12-54.png)



## 访问标志 access_flags

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-08_08-33-58.png)

access_flags中一共有16个标志位可以使用，当前只定义了其中9个 ，没有使用到的标志位要求一
律为零。

## “类”索引

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-08_08-37-00.png)

通过以上方式查找、解析全限定名。

## 字段表

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-08_08-53-47.png)

## 方法表

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-08_09-43-30.png)

## 属性表

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-08_09-52-38.png)

# 内存（方法区）中的class结构体