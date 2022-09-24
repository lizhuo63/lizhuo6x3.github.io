# 集合

### 集合的遍历方式

**list：** 普通for、增强for、list.forEach()、Iterator迭代器、stream.forEach()。**五种**。

**set：**增强for、set.forEach()、Iterator迭代器、set.stream().forEach。**四种**。

**map：**

+ 键值对遍历：map.entrySet() 加增强for、map.forEach((key,value) -> {})、map.entrySet().iterator()、map.entrySet().stream().forEach
+ 先键后值：map.keySet() 加增强for。**五种**。

**<u>都可以直接调用forEach()进行遍历。</u>**

### List去重方法

+ **Set**转手去收

  ```java
  List<String> newList = new ArrayList<>(new HashSet<>(list));//错序
  List<String> newList = new ArrayList<>(new TreeSet<>(list));//有序
  ```

+ **Stream流**distinct去重

  ```java
  list.stream().distinct().collect(Collectors.toList());
  ```

+ **新建一个List**，使用contains()判断添加

  ```java
  for (String cd:list) {
    if(!newList.contains(cd)){ //主动判断是否包含重复元素
    newList.add(cd);
  }
  ```
  
+ **对象去重**，重写equals() 和 hashCode()

  ```java
  class User implements Serializable{
  	str name;
  	int age;
  	@Override
  	public int hashCode() {
      return Objects.hash(name, age);
  	}
  	@Override
   public boolean equals(Object anObject) {
     if (this == anObject) {
         return true;
     }
     if (anObject instanceof User) {
     //根据业务判断
     }
  }
  List<User> newUserList = userList.stream().distinct().collect(Collectors.toList());
  ```



### 快速失败和安全失败

+ **fail-fast：**在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改就会抛出 Concurrent Modification Exception。 是java集合的一种错误检测机制，当多个线程对集合进行结构上的改变操作时，有可能会产生 fail-fast 机制。

  **解决办法**：加synchronized、使用CopyOnWriteArrayList来替换ArrayList

+ **fail-safe：**采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。



### HashMap的put流程

1. 计算hashcode【使用移位让hashcode的高低位进行异或运算将hashcode打散】，但是hashcode必须大于65536 (2^16) ，`(h = key.hashCode()) ^ (h >>> 16);`
2. 若map为null或桶数组长度为0则，初始化map
3. `i = (n - 1) & hash` 确定插入位置i
4. 若tab[i]=null，则将key,value封装为Node,直接存入tab[i]
5. 若tab[i] != null && tab[i] 为 **TreeNode**，则将kv封装为 TreeNode插入到红黑树中，若该key已存在，则在遍历过程中会直接覆盖旧值
6. 若tab[i] != null && tab[i] 为 **Node**，则将kv封装为 Node 插到链尾，若该key已存在，则在遍历过程中会直接覆盖旧值。插入后若链的长度达到了8，就尝试将该链进行树化，【它会判断当前map的键值对数量是否达到了64，如果是就树化，否则就扩容。】
7. 判断扩容

#### HashMap的扩容机制

**时机：** 当map中的键值对数量超过0.75容量时，就会对桶数组进行2倍扩容

**流程：**

1. 新容量和新的扩容阈值均 x2
2. 进行节点拷贝。在旧桶数组不为空的情况下，遍历数组
3. 单节点：重新hash映射到新数组中
4. 树节点：拆分树，并重新hash映射到新数组中
5. 链节点：拆为高低链，低链不变，高链的新索引为[ i + oldCap]。【拆分规则：(e.hash & oldCap) == 0，它的作用是拿到扩容前后取余的那个偏移位的bit，若为0，则扩容后索引位不会改变(余数不变)并将其加到**低链**；若为1，则扩容后索引位会在原位置增加"旧容量"(余数的高位加了一个1)并将其加到**高链**】

### HashMap 的桶长度为什么是2的幂次方

在创建HashMap时有两类方法，给容量和不给容量。

当不给容量时，hashmap设置16为初始容量，此后每次都两倍扩容，是2的幂次方。

当给容量时，它会调用 tableSizeFor(initialCapacity); 来调整hashmap的桶数组长度，向上最接近的2的幂次值。所以HashMap 的桶长度一定是2的幂次方。

**原因：** 为了保证hashMap的高效，需要hashcode足够分散，另外在插值或扩容时，需要频繁地确定插入位置，而 `i = (n - 1) & hash` 虽然可以保证性能，但是必须要求n为2次幂值。

### HashMap中K的类型如何选择

优先选择String、Integer等包装类。主要是能够保证Hash值的不可更改性和计算准确性，能够有效的减少Hash碰撞的几率。如果要使用自定义的对象就必须要重写hashCode()【**存储位置**】和equals()【**保证key在哈希表中的唯一性**】方法

### HashMap与HashTable的区别

1. HashMap线程不安全；Hashtable使用了synchronized，线程安全；
2. HashMap允许K/V都为null；后者K/V都不允许为null；
3. HashMap继承自AbstractMap类；而Hashtable继承自Dictionary类；

### ConcurrentHashMap和Hashtable的区别

+ ConcurrentHashMap 结合了 HashMap 和 HashTable 二者的优势。HashMap 没有考虑同步，HashTable 考虑了同步的问题。但是 HashTable 在每次同步执行时都要锁住整个结构。 ConcurrentHashMap 锁的方式是稍微细粒度的。

### HashMap 和 ConcurrentHashMap 的区别

1. ConcurrentHashMap对整个桶数组进行了分割分段(Segment)，然后在每一个分段上都用lock锁进行保护，相对于HashTable的synchronized 锁的粒度更精细了一些，并发性能更好，而HashMap没有锁机制，不是线程安全的。（JDK1.8之ConcurrentHashMap启了一种全新的方式实现,利用CAS算法。）
2. HashMap的键值对允许有null，但是ConCurrentHashMap都不允许。

### HashSet是如何保证数据不可重复的

HashSet的底层其实就是HashMap，只不过HashSet是把数据作为K值，而V值是都是一个相同值（虚值）来保存。而HashMap的K值是不允许重复的，所以HashSet的数据也就不可能重复了。



### 多线程场景下如何使用 ArrayList

`List<String> synchronizedList = Collections.synchronizedList(list);`

