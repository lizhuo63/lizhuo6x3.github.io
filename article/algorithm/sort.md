![](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/Snipaste_2022-06-16_00-41-04.png)

# 冒泡排序

每一趟都将两两相邻元素进行比较，大的值放后面

```java
    //两两相比，大的往上冒
    public static int[] maopaoSort(int[] arr) {
        int length = arr.length;
        //要比较n-1趟
        for (int i = 0; i < length - 1; i++) {
            //第i趟要比较的次数
            for (int j = 0; j < length - 1 - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    int c = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = c;
                }
            }
        }
        return arr;
    }
```



# 选择排序

默认以每一趟的开始元素为最小值，扫描后面的元素，记录比当前值还要小的元素位置，扫描结束后交换。

```java
 public static int[] xuanzeSort(int[] arr) {
        int length = arr.length;
        //当前最小值
        int minIndex,temp;
        //要比较n-1趟
        for (int i = 0; i < length - 1; i++) {
            minIndex=i;
            for (int j = i+1; j < length; j++) {
                if (arr[j] <arr[minIndex]) {
                    //扫描并记录比当前最小值更小的值的位置
                   minIndex=j;
                }
            }
            //交换arr[i]与arr[minIndex]
            temp = arr[i];
            arr[i] = arr[minIndex];
            arr[minIndex] = temp;
        }
        return arr;
    }

```



# 插入排序

将整个数组看做左边是已排序，右边待排序，挨个将待排元素与已排序数组从右到左的元素进行比较，小就交换。

```java
 public static int[] charuSort(int[] arr) {
        int length = arr.length;
        int temp=-1;
        //将0元素看作是排好序的，从1元素开始比较
        for (int i = 1; i < length; i++) {
            //将待排元素与它前一个元素进行比较，待排元素小就交换
            for (int j = i; j >0; j--) {
                if (arr[j] <arr[j-1]) {
                    temp = arr[j];
                    arr[j] = arr[j-1];
                    arr[j-1] = temp;
                }
            }
        }
        return arr;
    }
```

