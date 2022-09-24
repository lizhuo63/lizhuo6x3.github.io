# 设计模式

### 六大原则

+ **开放封闭原则：** 解决需求变化，要尽量通过扩展而不是修改原有代码
+ **里氏代换原则：** 基类可以随意替换子类，子类在继承或实现时不能修改父类或接口的方法
+ **依赖倒转原则：** 面向接口编程，方法传值时要尽量用层次高的类/接口
+ **接口隔离原则：** 尽量用多个独立、隔离的接口代替单个通用的接口

+ **迪米特法则：** 一个类要尽量减少自己对其他对象的依赖
+ **单一职责原则：** 一个方法只负责一件事情

### 单例模式

**目的：** 确保目标类的实例单一

**实现：**

1. 私有化构造方法，阻断外界随意创建的可能
2. 构造一个私有的静态实例引用
3. 提供以供公开的访问方法

**形式：** 

+ **饿汉式：**在类初始化时立即加载该对象，线程安全,调用效率高。
+ **懒汉式：** 只在真正需要使用的时候才会创建该对象，具备懒加载功能 [ synchronized ] 同步。
+ **静态内部类：** 结合了懒汉和饿汉各自的优点，实现了懒加载，而且线程安全。

+ **双重检测锁方式：** volatile ---> 防止指令重排 ；同步前判空：为了在单例存在时直接跳过，保证性能；同步后判空：为了安全创建单例对象，保证不重复创建。实现了懒加载，而且线程安全。
+ **枚举类：** 依据枚举类的构造只会调用一次，故将单例的实例化放到枚举的构造方法中。

### 工厂模式

#### 简单工厂

**实现方法：** 创建了抽象产品、具体产品和具体工厂，然后让具体工厂去依赖抽象产品，实现工厂与产品的解耦。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924095124949.png" alt="image-20220924095124949" style="zoom:67%;" />

**点评：** 虽然可以通过参数获取指定类型的对象，但是产生了抽象产品和具体产品间新的耦合，违反了开闭原则。

#### 工厂方法

**实现方法：** 创建抽象工厂、具体工厂、抽象产品和具体产品，然后**通过构造参数为用户引入具体工厂**，并且**匹配具体工厂和具体产品之间的依赖关系**就ok了。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924094417227.png" alt="image-20220924094417227" style="zoom:67%;" />

**点评：** 实现了用户与产品创建之间的解耦，但是在后期扩展中，需同时新增具体工厂和具体产品，会增加系统的复杂度。

#### 抽象工厂

解决产品族的生产问题

**实现方法：** 拥有和工厂方法一样的组件，只不过在工厂中声明了（套餐式）多种产品创建。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924152526226.png" alt="image-20220924152526226" style="zoom:67%;" />

**点评：** 在工厂方法之上提供了成套产品的创建方式，缺点与工厂方法一致。

### 原型模式

通过对象的拷贝【深拷贝需要使用对象流】，实现多个相同类型对象的简化创建。

**实现：** 创建一个具体原型类实现Cloneable接口、抽象原型接口可有可无（看需求）。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924153636579.png" alt="image-20220924153636579" style="zoom:67%;" />

```java
//奖状类
public class Citation implements Cloneable {
    private String name;
    public void setName(String name) {
        this.name = name;
    }
    public String getName() {
        return (this.name);
    }
    public void show() {
        System.out.println(name + "同学：表现优秀，特发此状！");
    }
    @Override
    public Citation clone() throws CloneNotSupportedException {
        return (Citation) super.clone();
    }
}
//测试访问类
public class CitationTest {
    public static void main(String[] args) throws CloneNotSupportedException {
        Citation c1 = new Citation();
        c1.setName("张三");
        //复制奖状
        Citation c2 = c1.clone();
        //将奖状的名字修改李四
        c2.setName("李四");
        c1.show();
        c2.show();
    }
}
```

### 建造者模式

用于便捷创建含有多组件的复杂对象。

* 抽象建造者类（Builder）：提供所有独立组件的set方法，和最终产品的创建方法；相当于一个空白的组件配置表。【创建空白表】

* 具体建造者类（ConcreteBuilder）：实现 Builder 接口，添加特定产品的组件配置。相当于在组件配置表中填写了具体的组件信息。【填表】

* 产品类（Product）：具体的创建对象。

* 指挥者类（Director）：依赖了一个Builder [配置单]，并提供了最终产品的创建方法。

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220924163656877.png" alt="image-20220924163656877" style="zoom:67%;" />

上述的具体建造者类