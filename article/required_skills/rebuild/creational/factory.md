# 传统策略：ifelse匹配

​		在未涉及模式之前，如果有大量构建实例的需求，通常都会采取 ifelse 匹配的手段来实现。

```java
if(i=1){
		new A();
}else if(i=2){
		new B();
}
...
else{
		new N();
}

public A(){}
public B(){}
...
public N(){}
```

​		但是这种方法会导致创建策略于创建逻辑的强耦合，后期当需求足够大时，不仅要更新插件策略还要添加创建逻辑，此外它们封到同一个类中，极其不宜维护。是该换换当家的了😡。



# 简单工厂

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-29_07-28-42.png)

如上，我们将创建逻辑进行分层实现，这样就可以将常见逻辑给分离出来，但是他有明显的缺陷，即用户传入的是同一份参数，SimpleFactory并不知道用户需要哪一种特定的产品，所以我们还是不得不使用 ifelse匹配手段 来实现创建策略。

```java
/**
 * 解析：客户委托工厂生产产品，仅需提供产品信息{大众标准【产品接口】，个性配置【参数列表】}
 *      工厂负责根据图纸【接口】，特殊要求【参数列表】完成具体产品的实现createXXX，
 *      缺陷；新增产品时要通过creatFruit[耦合判断]，不方便
 */
public class SimpleFactory {
    public static Fruit creatFruit(String name) {
        if (name.equals("watermelon")) {
            return new Watermelon();
        } else if (name.equals("watermelon")) {
            return new Orange();
        } else {
            throw new IllegalArgumentException("不支持参数" + name);
        }
    }
//===================================================
    //产品规格
    interface Fruit {
        void eat();
    }
//===================================================
    //产品实现
    public class Watermelon implements Fruit {
        @Override
        public void eat() {
            System.out.println("解渴");
        }
    }
//===================================================
    //产品实现
    public class Orange implements Fruit {
        @Override
        public void eat() {
            System.out.println("酸");
        }
    }

    //产品测试
    public static void main(String[] args) {
        Fruit watermelon = SimpleFactory.creatFruit("watermelon");
        watermelon.eat();
    }
}
```

​		可以看到，还是需要引入标志参数，对其动态匹配。所以这种工厂是一种很垃圾的工厂，因为它完全没有达到我们的预期，基本上是不会使用这种垃圾的。



# 工厂方法

​		方才说到普通工厂存在需求不符的致命短板，主要是由于工厂无法感知用户的需求，无法满足在不传标志参数的条件下动态匹配目标需求产品。如果使用多态机制呢，参考产品层的设计，我们也对工厂进行实现分层，让下层工厂一对一匹配目标需求，貌似问题就迎刃而解了。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220629074619999.png)

```java
public class FactoryMethod {
    //规格
    interface Phone {
        void call();
    }
//================================================
    //产品实现
    private static class  XiaoMi implements Phone {
        @Override
        public void call() {
            System.out.println("小米呼叫");
        }
    }
//================================================
    public class HuaWei implements Phone {
        @Override
        public void call() {
            System.out.println("华为呼叫");
        }
    }
//================================================
    //工厂
    interface PhoneFactory {
        Phone createPhone();
    }
//================================================
    //具体工厂实现
    public class XiaoMiFactory implements PhoneFactory {
        @Override
        public Phone createPhone() {
            return new XiaoMi();
        }
    }
//================================================
    //具体工厂实现
    public class HuaWeiFactory implements PhoneFactory {
        @Override
        public Phone createPhone() {
            return new HuaWei();
        }
    }

    public static void main(String[] args) {
        Phone phone = new XiaoMiFactory().createPhone();
        Phone phone1 = new HuaWeiFactory().createPhone();
        phone1.call();
        phone.call();

    }
}
```

​		如此，创建两个接口【工厂接口、产品接口】，并分头实现，就能达到两端一对一连线，动态匹配指定产品的需求了，问题解决👍。



# 抽象工厂

​		动态匹配创建的需求已经实现了，可以说工厂方法可以实现所有单产品的创建，这里指定的单产品是没有子产品实例的说法。那么现在如果要造枪造炮去投入战斗，那就有一点不同了。

1. 首先武器应该是成套的，用户虽然没有提这个需求，但是我们绝对不能将枪身和炮弹一起打包发给用户，用不了啊
2. 我们希望匹配打包的过程能够自动识别，也就是说不由用户来选择匹配，也许用户会传入武器型号，那么我们就要通过型号来连线匹配，总之，要让用户用得爽。

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-29_13-54-33.png)

```java
public interface DrinksFactory {
    Water createWater();
    Bottle createBottle();
}
//=========================
public class ColaFactory implements DrinksFactory{
    @Override
    public Water createWater() {
        return new Cola();
    }
    @Override
    public Bottle createBottle() {
        return new RedBottle();
    }
}
//=========================
public class SpriteFactory implements DrinksFactory{
    @Override
    public Water createWater() {
        return new Sprite();
    }
    @Override
    public Bottle createBottle() {
        return new GreenBottle();
    }
}
//=====================
public interface Water {
    String getWater();
}
//===================
public class Cola implements Water {
    @Override
    public String getWater() {
        return "可乐";
    }
}
//=======================
public class Sprite implements Water {
    @Override
    public String getWater() {
        return "雪碧";
    }
}
//======================
public interface Bottle {
    String getBottle();
}
//=====================
public class GreenBottle implements Bottle{
    @Override
    public String getBottle() {
        return "绿瓶子";
    }
}
//====================
public class RedBottle implements Bottle {
    @Override
    public String getBottle() {
        return "红瓶子";
    }
}
```

​		抽象工厂接口就好似一个可有可无的门面，主要还是由他的下层实现类进行各个组件的拼装，已达到创建成配套实例的需求，日后若是要推出新产品，只需要创建一个新的实现类并附加新的组合即可。



# 总结

这三个工厂的初衷就是实现解耦创建实例，达到日后新增时能便利的维护。

+ **简单工厂** 只是将创建策略于创建逻辑分离开了，仍旧未能摆脱 ifelese匹配的强耦合，基本上就是个垃圾，网上有关工厂模式通常没有简单工厂，这都是有原因的。
+ **工厂方法** 首次实现了创建实例的解耦，采取两端实现，利用多态机制成功摆脱了ifelse匹配的耦合。好用。
+ **工厂方法** 如果我们的需求产品是组件套组件，有拼装的场景，那么工厂方法就有一些不好使了。于是工厂方法诞生了，说白了就是在工厂实现中实现组装策略。





















































