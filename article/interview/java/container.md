
# 集合

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

**对象去重**，重写equals() 和 hashCode()

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
## 集合

### HashMap 的⻓度为什么是2的幂次⽅

HashMap在定位桶索引时，使用了位运算进行了优化【hash%length==hash&(length-1)】，但该式子成立的前提就是 **length是2的n次⽅**。

### 快速失败(fail-fast)和安全失败(fail-safe)

+ **fail-fast：**在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改就会抛出 Concurrent Modification Exception。
+ **fail-safe：**采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原
  有集合内容，在拷贝的集合上进行遍历。

