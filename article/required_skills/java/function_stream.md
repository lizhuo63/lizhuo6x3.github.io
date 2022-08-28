# 核心思想

+ 面向对象的思想：实现一个需求需要使用那个类。 

+ 函数式的思想：实现一个需求，我需要进行哪些操作。

可见，对于繁琐的步骤调用场景，函数式的更加间接高效，它是将面向功能【形式上，通过将代码块作为参数来传值】。



# Lambda表达式

给人的直观感受是以函数为参数，极好地避免了大量对象的声明。多用于匿名内部类的语法简化以及函数式接口【只包含一个抽象方法的接口】。使用原则：【可推到可省略，即若方法列表可以推到出来就可以省略方法列表；同样可应用到方法名】`(参数列表)->{方法体}`。

那又该如何理解它呢？Lambda只支持函数式接口，所以**一旦你使用了该接口的功能，jvm就能推断你使用的具体函数，以及相应的返回值。**Lambda也是基于此来实现对匿名内部类进行代码简化的。如下：

`BinaryOperator<Long> add = (x, y) -> x + y;` 不应理解为将值相加返给add。而是在此处，要与接口，方法挂钩。**正解：在 BinaryOperator 接口特定的某个方法中 插入了等号右侧的方法体，并为这整个方法实现提取名为 add。**故而add代表的是一个方法，在接口层来看，将一个参数列表传给了接口，接口经过处理后返回了结果，基于这个角度也可已将add理解为一个功能器件。

## 常用的函数式接口

+ **Supplier**：生产一个泛型数据

  ```java
  //定义一个方法返回一个整数数字
      private static Integer getInteger(Supplier<Integer> sup) {
          return sup.get();
      }
    public static void main(String[] args) {
          Integer i = getInteger(() -> 555);
          System.out.println(i);
      }
  ```

+ **Consumer**：链式消费一个数据【**强调：消费不同于数据操作，无法对数据源进行链式更改，每一个环节都是对同一份数据源进行操作**】

  + andThen：执行链式的消费并返回Consumer,【无法生产新的数据，一整个链都是消费的同一份数据源】

  + accept：执行消费并返回void

    ```java
    default Consumer<T> andThen(Consumer<? super T> after) {
            Objects.requireNonNull(after);
            return (T t) -> { accept(t); after.accept(t); };
        }
    ```

    ```java
     public static void main(String[] args) {
            operatorString("abc",s ->new StringBuffer(s).append("de"), s-> System.out.println(s+"f"));
        } // abcf
    
        private static void operatorString(String name, Consumer<String> con1, Consumer<String>con2){
            con1.andThen(con2).accept(name);//con1和con2先后对name进行消费
        }
    ```

+ **Predicate**：断言，判断是非

+ **Function**：类型转化

  + andThen：组合链式操作
  + apply：终结链式操作，并返回结果

  ```java
  public static void main(String[] args) {
          convert("abc,18",s -> s.split(",")[1],s -> Integer.parseInt(s),
                  i->i+70);//88
      }
  
      private static void convert(String s, 
                                  Function<String,String> fun1,//类型转换
                                  Function<String,Integer>fun2,//类型转换
                                  Function<Integer,Integer>fun3){//同型操作
          System.out.println(fun1.andThen(fun2).andThen(fun3).apply(s));
      }
  ```



## 方法引用

标准语法为 `Classname::methodName` 。



# Stream流

Stream流是对集合等可迭代对象进行迭代处理的一种优化，借助lambda表达式将迭代逻辑从业务中实现置换到有Stream对象来迭代。

![image-20220818000534032](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220818000534032.png)

## 获取 Stream流 的方式

1. **集合.stream()**
2. **集合.parallelStream()**【并行流】
3. **Stream.of(序列)**

## 常用API

**惰性求值操作：返回一个新的Stream流，利于链式操作**

+ collect：挑选当前流中的元素到新的流中

  ```java
  // 将元素放到自定义的集合中
  stream.collect(toCollection(TreeSet::new));
  ```

  自定义Collector收集器

+ concat：生成一个二合一的新流

+ filter：断言过滤流中的元素

+ distinct：元素去重

+ flatMap：将当前流中子一级的所有集合中的元素全部取出，放到一个新的流中

+ map：更新流中的元素【以新换旧】，如`.map(string -> string.toUpperCase()) `

+ limit：取出当前流中的前n个元素

+ skip：跳过前n个元素，与limi效果互补

**及早求值操作：返回一个非Stream流的对象，终结链式操作**

+ allMatch：断言所有元素
+ anyMatch：断言任意元素
+ max：返回最大元素
+ min：返回最大元素
+ forEach：对每一个元素进行消费
+ toArray：将所有元素存入数组
+ sorted：将流中的元素进行排序
+ reduce：对流中的元素递归的执行二元操作，实现从一组值中生成一个值。（如简化阶乘操作）

**flatMap示例**

```java
public static void main(String args[]) {

		List<String> teamIndia = Arrays.asList("Virat", "Dhoni", "Jadeja");
		List<String> teamAustralia = Arrays.asList("Warner", "Watson", "Smith");
		List<String> teamEngland = Arrays.asList("Alex", "Bell", "Broad");

		List<List<String>> playersInWorldCup2016 = new ArrayList<>();
		playersInWorldCup2016.add(teamIndia);
		playersInWorldCup2016.add(teamAustralia);
		playersInWorldCup2016.add(teamEngland);

		List<List<String>> playersInWorldCup2017 = new ArrayList<>();
		playersInWorldCup2017.add(teamIndia);
		playersInWorldCup2017.add(teamAustralia);
		playersInWorldCup2017.add(teamEngland);

		List<List<List<String>>> playersInWorldCup = new ArrayList<>();
		playersInWorldCup.add(playersInWorldCup2016);
		playersInWorldCup.add(playersInWorldCup2017);

		playersInWorldCup.stream().flatMap(w->w.stream()).flatMap(a->a.stream()).forEach(strings -> System.out.println(strings));

	}
```

## 【一行挑战】功能演示

**数据源**

```
public static void main(String[] args) {
		Integer[] ints = {5, 9, 7, 3, 6, 4, 8, 2};
	}
```

### 排序

```java
//数组
Arrays.stream(ints).sorted((a, b) -> a - b).collect(Collectors.toList()).forEach((s) -> System.out.println(s));

```









