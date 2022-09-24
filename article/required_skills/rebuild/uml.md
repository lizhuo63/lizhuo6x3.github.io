# 类的表示方式

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924072554324.png" alt="image-20220924072554324" style="zoom:67%;" />

UML类图中表示可见性的符号有三种：

- +：表示public
- -：表示private
- \#：表示protected

属性的完整表示方式是： **可见性 名称 ：类型 [ = 缺省值]**

方法的完整表示方式是： **可见性 名称(参数列表) [ ： 返回类型]**



# 类与类关系表示

## 关联（has-a）

### 单向关联

每个顾客都有一个地址

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924072841429.png" alt="image-20220924072841429" style="zoom:67%;" />

### 双向关联

互相持有

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924073002192.png" alt="image-20220924073002192" style="zoom:67%;" />

### 自关联

持有子节点

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924073100038.png" alt="image-20220924073100038" style="zoom:67%;" />

## 聚合（has-a，可分离）

成员对象是整体对象的一部分，但是成员对象可以脱离整体对象而独立存在。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924073305428.png" alt="image-20220924073305428" style="zoom:67%;" />

## 组合（has-a，不可分离）

整体对象可以控制部分对象的生命周期，一旦整体对象不存在，部分对象也将不存在，部分对象不能脱离整体对象而存在。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924073431936.png" alt="image-20220924073431936" style="zoom:67%;" />

## 依赖（use-a）

对象之间**耦合度最弱**的一种关联方式，是临时性的关联。在代码中体现为，由类的方法通过局部变量、参数列表或静态方法来**调用访问被依赖类的某些方法**来完成一些职责。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924074021679.png" alt="image-20220924074021679" style="zoom:67%;" />

## 继承（is-a）

继承关系是对象之间耦合度最大的一种关系，是父类与子类之间的关系。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924074208272.png" alt="image-20220924074208272" style="zoom:67%;" />

## 实现（is-a）

接口与实现类之间的关系

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924074255198.png" alt="image-20220924074255198" style="zoom:67%;" />



# 六大设计原则

+ **开放封闭原则：** 代码迭代时，要尽量通过扩展而不是修改原有代码
+ **里氏代换原则：** 基类可以随意替换子类，子类在继承或实现时不能修改父类或接口的方法
+ **依赖倒转原则：** 面向接口编程，方法传值时要尽量用层次高的类/接口
+ **接口隔离原则：**一个类对另一个类的依赖应该建立在最小的接口上。 尽量用多个独立、隔离的接口代替单个通用的接口

+ **迪米特法则：** 一个类要尽量减少自己对其他对象的依赖
+ **单一职责原则：** 一个方法只负责一件事情