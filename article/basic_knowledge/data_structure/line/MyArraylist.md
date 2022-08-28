```java
/**
 * 自实现变长版
 *
 * 理清思路：list的容器就是数组，所以数组的长度就是容量，
 * 分清楚N与arr.length的区别：N元素个数，由于支持null值arr.length代表容器的容量，所以arr.length>=N
 * @author lizhuo
 */
public class MySequenceList<E> implements Iterable {

    /**
     * 容器数组
     */
    private E[] arr;

    /**
     * 元素个数,全局变量
     */
    private static int N;

    /**
     * 自主构造
     * @param capacity 容量
     */
    public MySequenceList(int capacity) {
        //数组的长度即为list的容量
        arr = (E[]) new Object[capacity];
        N = 0;
    }

    /**
     * 默认构造
     */
    public MySequenceList() {
        arr = (E[]) new Object[10];
        N = 0;
    }

    /**
     * 空判断
     * @return
     */
    public boolean isEmpty() {
        return N == 0;
    }

    /**
     * 获取元素个数
     * @return
     */
    public int size() {
        return N;
    }

    /**
     * 获取指定位置的元素
     * @param i 目标索引
     * @return
     */
    public E get(int i) {
        if (i < 0 || i > arr.length) {
            throw new RuntimeException("索引越界");
        }
        return arr[i];
    }

    /**
     * 查出t第一次出现的位置
     * @param e 查询元素
     * @return
     */
    public int firstIndex(E e) {
        if (e == null) {
            throw new RuntimeException("不支持null搜索");
        }
        int result = -1;
        for (int i = 0; i < N; i++) {
            E it = arr[i];
            if (e.equals(it)) {
                result = i;
            }
        }
        //无此元素就返回一个不存在的索引
        return result;
    }

    /**
     * 查出t最后出现的位置
     * @param e 目标元素
     * @return
     */
    public int lastIndex(E e) {
        if (e == null) {
            throw new RuntimeException("无此元素!");
        }
        int result = -1;
        for (int i = N; i > 0; i--) {
            E it = arr[i];
            if (e.equals(it)) {
                result = i;
            }
        }
        return result;
    }

    /**
     * 在尾部添加元素
     * @param e 添加元素
     */
    public void add(E e) {
        if (N == arr.length) {
            resize();
        }
        arr[N++] = e;
    }

    /**
     * 在指定位置插入元素
     * @param i 目标位置
     * @param e 新元素
     */
    public void add(int i, E e) {
        if (N == arr.length) {
            resize();
        }
        if (i < 0 || i > arr.length) {
            throw new RuntimeException("插入位置不合法!");
        }
        //将i及其后元素后移一位
        if (N - i >= 0) {
            System.arraycopy(arr, i, arr, i + 1, N - i);
        }
        arr[i] = e;
        N++;
    }

    /**
     * 删除所有元素
     */
    public void clear() {
        for (int i = 0; i < N; i++) {
            arr[i] = null;
        }
        N = 0;
    }

    /**
     * 删除指定位置的元素
     * @param index 目标位置
     * @return 被删除的元素
     */
    public E remove(int index) {
        if (index < 0 || index > N - 1) {
            throw new RuntimeException("指定位置不合法!");
        }
        E result = arr[index];
        //将i后的元素前移一位
        if (N - index >= 0) {
            System.arraycopy(arr, index + 1, arr, index, N - index);
        }
        //删除尾元素
        arr[N - 1] = null;
        N--;
        resize();
        return result;
    }

    /**
     * 更改目标位置下的值
     * @param index 目标位置
     * @param t 新元素
     * @return 删除状态
     */
    public boolean update(int index, E t) {
        boolean result = false;
        if (index < 0 || index > N - 1) {
            throw new RuntimeException("索引越界");
        } else {
            arr[index] = t;
            result = true;
        }
        return result;
    }

    /**
     * 调整集合的容量
     */
    public void resize() {
        int newSize = 0;
        int oldSize = arr.length;
        //扩容
        if (N == arr.length) {
            newSize = oldSize + (arr.length >> 1);
            arr = (E[]) Arrays.copyOf(arr, newSize);
        }
        //容量缩小
        if (arr.length > (N << 1) - 1) {
            newSize = oldSize * 3 / 4;
            arr = Arrays.copyOf(arr, newSize);
        }
    }

    @Override
    public Iterator<E> iterator() {
        return new Iterator() {
            private int cur;
            @Override
            public boolean hasNext() {
                //指针位置小于元素个数,说明有下一个值
                return cur < N;
            }
            @Override
            public Object next() {
                return arr[cur++];
            }
        };
    }
}

```

