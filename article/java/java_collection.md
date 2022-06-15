# Collection

## 详情

数组和集合的区别:

- 长度的区别
  - **数组的长度固定**
  - **集合的长度可变**

- 内容不容
  - 数组存储的是同一种类型的元素
  - 集合可以存储不同类型的元素(但是一般我们不这样干..)

- 元素的数据类型
  - 数组可以存储基本数据类型,也可以存储引用类型
  - **集合只能存储引用类型(你存储的是简单的int，它会自动装箱成Integer)**

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-05_07-20-42.png)

## Iterator

iterator( )返回的**Iterator**也是一个接口，它只有三个方法：

- hasNext() 判断
- next() 遍历
- remove() 移除

```java
private class Itr implements Iterator<E> {
	//游标
    int cursor = 0;
	//上一次迭代到的游标，每次使用完就会重置为 -1
    int lastRet = -1;
	//用来检查是否发生了并发操作，如果这两个值不一致，就会报错
    int expectedModCount = modCount;//此List被修改的次数

    public boolean hasNext() {
        return cursor != size();
    }

    public E next() {
        checkForComodification();
        try {
            int i = cursor;
            E next = get(i);//调用子类的方法，继承AbstractList需要实现size(),get()两个方法。
            lastRet = i;//前置元素的游标
            cursor = i + 1;//当前游标
            return next;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();
        try {
            AbstractList.this.remove(lastRet);//自身remove()为空实现，调用需要子类的remove()方法
            if (lastRet < cursor)
                //删除成功后，游标会回退1
                cursor--;
            lastRet = -1;//上一次的索引对应元素已不存在，将其重置为-1
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException e) {
            throw new ConcurrentModificationException();
        }
    }
    
	//当实际修改次数与预期修改次数不符(发生了并发并发操作)时，抛出并发异常，每次next(),remove()都会做此检查
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

# List

## 特点

存取有序，元素可重复、有索引，支持null

## ListIterator

在**Iterator**基础上添加了向前遍历、添加元素、设值等功能。

```java
private class ListItr extends Itr implements ListIterator<E> {
    //多了一个可从指定索引迭代的构造
    ListItr(int index) {
        cursor = index;
    }
	//检查是否有前置元素
    public boolean hasPrevious() {
        return cursor != 0;
    }
		
    public E previous() {
        checkForComodification();
        try {
            int i = cursor - 1;
            E previous = get(i);
            lastRet = cursor = i;//记录上前置元素的游标
            return previous;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }
	
    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor-1;
    }

    public void set(E e) {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();
        try {
            AbstractList.this.set(lastRet, e);
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    public void add(E e) {
        checkForComodification();
        try {
            int i = cursor;
            AbstractList.this.add(i, e);
            lastRet = -1;
            cursor = i + 1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}
```

## ArrayList(异步数组)

**特点**

- ArrayList的默认初始化容量是10，
- **添加元素**时，首先确定所需的最小容量，然后判断最小容量是否够用（够用直接添加），来决定是否执行1.5倍扩容，随后copyof( )拷贝元素。
- **删除**元素时，首先检查索引，删除元素后，整体左移，并将末尾元素设null，GC回收。删除操作不会改变容器的容量，可通过**trimToSize()**手动调整获取最小容量。
- 在**增删**时候，需要数组的拷贝复制(navite 方法)

**性能**

+ ArrayList查询效率高，增删慢

## LinkedList(异步双向链表)

**特点**



**性能**



## Vector(同步数组)

+ Vector所有方法都是同步的，**有性能损失**。
+ Vector的初始length是10 
+ Vector的扩容倍率为2，相比ArrayList会更耗内存

- 

# Set

## 特点

存取无序、唯一，支持null

## HashSet

+ 本质上是一个实现了Set接口的HashMap实例，它的键为泛型，值是一个虚拟的new Object( )。

    ​	![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_09-23-33.png)

## TreeSet

+ 本质上是一个实现了Set接口的TreeMap实例，它的键为泛型，值是一个虚拟的new Object( )。

+ **可实现排序**

    ​	![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_09-34-46.png)

## LinkedHashSet

+ 本质上是一个继承了HashSet的LinkedHashMap实例

+ **存取有序**，性能比HashSet差一些，因为要维护一个双向链表

    ![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_09-43-46.png)

# Map

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-05_10-47-29.png)

## HashMap

+ 存取无序，异步散列表(哈希表)，支持null(单null键，多null值)。
+ 当初始容量(16)*装载因子(0.75)小于哈希表容量(实际数据量)时，会执行再散列(桶数2倍扩容)。最大容量为2^31

+ 当桶内元素不足6时，会优化为链表；当链表长度超过8时会调用treeifyBin方法，当桶数不足64时会执行扩容，达到64时会将链表优化为树。

+ 四大构造

    ![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-05_11-07-56.png)

+ **put操作**

    1. 首先通过key计算hash值，再调用putVal(hash,key,value,...)。
    2. 判断table（散列表）是否为空,长度是否为0。若真，则首先初始化table。
    3. 若发生哈希碰撞，则直接覆盖旧值，并返回旧值
    4. 若为树结构就调用树插入方法；若为链表结构，则遍历判断key是否已存在，有就break，否则添加到尾链。
    5. 返回新值
    6. 判断是否需要结构优化（树链转化）
    7. 判断扩容

    <img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-05_16-55-02.png"  />

    ```java
    // transient Node<K,V>[] table为哈希桶数组;
    // Node实现了Map.Entry接口，本质就是一个映射(键值对)
    // 参数onlyIfAbsent表示是否替换原值
    // 参数evict主要用来区别通过put添加还是创建时初始化数据的，可以忽略
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                    boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 桶数组(散列表)为空，需要初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            // resize()不仅用来调整大小，还用来进行初始化配置
            n = (tab = resize()).length;
        // (n-1) & hash是hash % n 高效的与运算优化，用于计算节点在桶数组中的存储位置，此处是在判断目标桶是否为空
        if ((p = tab[i = (n - 1) & hash]) == null)
            // 如果桶是空的，就将元素直接插进去
            tab[i] = newNode(hash, key, value, null);
        else {
            //这时就需要链表或红黑树了
            // e指定为待插入的元素
            Node<K,V> e; K k;
            // p是存储在目标位置的元素
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                //若hash值和key均相同。则拒绝插入，使用原值覆盖
                e = p; 
            // 若p是一个树节点
            else if (p instanceof TreeNode)
                // 把节点添加到树中
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 若p为链表结构，要把待插入元素挂在链尾
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 若链表较长，则需要树化；
                        // 初始状态为p.next，故到第8个元素才会被树化
                        if (binCount >= TREEIFY_THRESHOLD - 1)
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 遍历链表后，若key已存在，则不做处理。
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    // 若key不存在，就将新节点指向链尾位置
                    p = e;
                }
            }
            // 返回更改后的新值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                // 默认为空实现，允许我们修改完成后做一些操作
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 若size达到了capacity的0.75，则需要扩容
        if (++size > threshold)
            resize();
        // 默认也是空实现，允许我们插入完成后做一些操作
        afterNodeInsertion(evict);
        return null;
    }
    ```

+ **get操作**

    1. 计算key的hash值，调用getNode( )方法
    2. 若为桶中的首个元素则直接返回
    3. 否则遍历红黑树或链表来获取

+ **remove操作**

    1. 计算key的hash值，调用removeNode( )方法
    2. 首先判断桶不为空且hash值存在，若假则返回null
    3. 若桶的首位即目标，就记录该节点，否则就到树或链表中遍历查找
    4. 找到目标后，分链表、树、桶首位三种情况去删除

## HashTable

+ 不支持null值null键

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-05_18-51-51.png)

## LinkedHashMap

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-05_19-04-53.png)

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-05_19-06-55.png)

+ 在继承HashMap的基础之上，还维护了一个双向链表，保证了元素的有序性（存取有序，最近访问排序）。即LinkedHashMap=HashMap+双向链表
+ 存取有序，支持null，异步

### 成员变量

- K key === 继承自HashMap
- V value ===  继承自HashMap
- Entry<K, V> next ===  继承自HashMap，用于维护table（散列表）上Entry的顺序
- int hash ===  继承自HashMap
- **Entry<K, V> before**  ===  前继指针，维护Entry插入的先后顺序(维护双向链表)。
- **Entry<K, V> after** ===  后继指针，维护Entry插入的先后顺序(维护双向链表)。

### 初始化

```java
void init() {
    //初始化了一个Header,双向链表的链表头
    header = new Entry<K,V>(-1, null, null, null);
    header.before = header.after = header;
}
```

### 构造

```java
// 调用HashMap的对应构造，默认使用存取顺序排序
public LinkedHashMap() {}
public LinkedHashMap(int initialCapacity, float loadFactor) {}
public LinkedHashMap(int initialCapacity) {}
public LinkedHashMap(Map<? extends K, ? extends V> m) {}
// 选择使用最近访问排序
public LinkedHashMap(int initialCapacity,float loadFactor,boolean accessOrder) {}
```

### put

linkedhashmap使用的是hashmap的put方法，并且复写了newnode方法（新节点接入双向链表尾部）和afternodeaccess方法（新节点接入单链表尾部），以此实现它特有的put。

### get

在最近访问排序下，会将每次访问的元素排到链尾。**常用的放在链尾，不常用的放链头**

### remove

也是直接使用HashMap的方法来完成的，LinkedHashMap重写了afterNodeRemoval(Node<K,V> e)这个方法。

### 遍历

初始容量对遍历没有影响，它遍历的是内部的双向链表



## TreeMap

+ 异步红黑树，可排序(Comparable)

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_08-01-43.png)

### 构造

可通过Comparator实现自主排序，若没有实现comparator则使用自然顺序

![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_08-36-13.png)

### put

1. 判断key和红黑树是否为空，并进行初始化

2. compare比较，定位插入位置分为有无实现Comparator两种情况

3. 调整平衡红黑树的结构

    ```java
    public V put(K key, V value) {
               //用根节点t表示二叉树
                Entry<K,V> t = root;
                if (t == null) {
                    compare(key, key); // type (and possibly null) check
                    //初始化树
                    root = new Entry<>(key, value, null);
                    size = 1;
                    //修改次数 + 1
                    modCount++;
                    return null;
                }
                int cmp;     //cmp表示key排序的返回结果
                Entry<K,V> parent;   //父节点
                // split comparator and comparable paths
                Comparator<? super K> cpr = comparator;    //指定的排序算法
                //若自定义有comparator的实现
                if (cpr != null) {
                    do {
                        //parent指向上次循环后的t
                        parent = t;      
                        //比较新增节点和当前节点key的大小
                        cmp = cpr.compare(key, t.key);
                        //位置调整
                        if (cmp < 0)
                            t = t.left;
                        else if (cmp > 0)
                            t = t.right;
                        //若两个key值相等，则新值覆盖旧值，并返回新值
                        else
                            return t.setValue(value);
                    } while (t != null);
                }
                //若cpr为空，即采用默认的自然排序
                else {
                    if (key == null)
                        throw new NullPointerException();
                    //令key具备可比性，key需要实现Comparable
                    Comparable<? super K> k = (Comparable<? super K>) key;
                    do {
                        parent = t;
                        cmp = k.compareTo(t.key);
                        if (cmp < 0)
                            t = t.left;
                        else if (cmp > 0)
                            t = t.right;
                        else
                            return t.setValue(value);
                    } while (t != null);
                }
                //将新增节点当做parent的子节点
                Entry<K,V> e = new Entry<>(key, value, parent);
                //位置调整
                if (cmp < 0)
                    parent.left = e;
                else
                    parent.right = e;
                /*
                 *  至此，新增节点已插入到该树中的合适位置
                 *  下面的fixAfterInsertion()是对这棵树进行调整、平衡
                 */
                fixAfterInsertion(e);
                //TreeMap元素数量 + 1
                size++;
                //TreeMap容器修改次数 + 1
                modCount++;
                return null;
            }
    ```

### 遍历

可以使用map.values(), map.keySet()，map.entrySet()，map.forEach()

## ConCurrentHashMap

+ 支持高并发的访问和更新
+ **不支持null值和null键**，如果`ConcurrentHashMap.get(key)`得到了 null ，就无法判断value是null 还是无此key而为 null ，就有了二义性。
+ 检索操作不用加锁，get方法非阻塞

**HashTable**

只有一把表锁，并发效果极差

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_17-47-39.png" style="zoom:67%;" />

**JDK1.7  ConcurrentHashMap**

+  由Segment数组和HashEntry数组联合构成

+ 默认16个"桶"，16把分段锁，提高了并发能力
+ **特点：**将数据分段存储，并为每一段数据配一把可重入锁（由Segment继承ReentrantLock），当线程a占锁访问数据段A时，不会干扰其他线程对其他区段数据的访问

<img src="https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_17-49-27.png" style="zoom:67%;" />

**JDK1.8  ConcurrentHashMap**

+ 散列表+红黑树（同HashMap）

+ 采用[CAS](article/java/lock.md) + synchronized实现更加细粒度的锁，将锁设在哈希桶数组元素上，只需要锁住链头（树根），效能更优

  ![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-06_17-50-01.png)

### put

1. 根据key计算hashcode；
2. 判断是否需要初始化；
3. 定位元素的插入的目标桶f，并对桶(首节点)f进行判断
   1. 如果f为null ，则通过 CAS 的方式尝试添加；
   2. 若f.hash = MOVED = -1 ，说明有其他线程在扩容，则参与一起扩容；
   3. 若都不满足 ，就synchronized锁住f节点，遍历插入（判断链表、红黑树）；
4. 当在链表长度达到 8 的时候，数组扩容或者将链表转换为红黑树（桶数达到64）。

### get

get 方法不需要加锁。因为Node的值value 和指针next是用volatile修饰的，在多线程环境下线程A修改节点的 value 或者新增节点的时候是对线程B可见的。十分的高效。


