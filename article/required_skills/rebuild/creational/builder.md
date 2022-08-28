# 背景

​		一项需求可分化为诸多个小需求，每个小需求都有不同的实现，导致该需求有非常多的组合，所以直接 new() 将导致代码中出现大量的 setXxx() ,而且复用性很差，一旦需求变动，将是批量性的修改，不易维护。

# 思想

​		虽然产品有诸多小需求，但我们不关注这些小的需求项，我们直接将目光放到需求表这整张表上，再把这张表交给下层建造者去实现即可，总而言之，就是不能让产品自己new()或者set()。而是让第三方的建造来配置属性方案生成配置模板，然后由产品直接根据模板来构造。

# 实现方案

​		建造者多用于细粒度需求较多的场景，故一般都会采用链式调用。

## 类关联法(自命名)

​		直接由产品类关联一个建造者类，通过创建者项产品传参实现。由于没有继承和实现，即 Builder是一个普通类，导致复用性仍然很低，如果有其他模板方案将要继续重构。但是如果使用默认的配置方式 ` this.cpu = builder.getCpu();`就基本上能应对大部分的需求量，而且代码量少，结构清晰。

特点：

+ 结构简单，易维护。

+ 没有模板，要求随用随建。

**给出一个产品类：**

```java
public class Computer {
    private String cpu;
    private String ram;
    private String keyboard;

// 通过建造者的清单来创建，屏蔽无参构造
   public Computer(Builder builder) {
       this.cpu = builder.getCpu();
       this.keyboard=builder.getKeyboard();
       this.ram=builder.getRam();
   }
}
```

**给出建造者类：**

```java
public class Builder {
    private String cpu;
    private String ram;
    private String keyboard;

    public Builder setCpu(String cpu) {
        this.cpu = cpu;
        return this;
    }

    public Builder setRam(String ram) {
        this.ram = ram;
        return this;
    }

    public Builder setKeyboard(String keyboard) {
        this.keyboard = keyboard;
        return this;
    }

    public Computer build() {
        return new Computer(this);
    }
}
//=================调用方式
Computer ca = new Builder().setCpu("cpua").setKeyboard("kba").setRam("rama").build();
```



## 正规打法









