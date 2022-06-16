# 二分查找

## 顺序数组

```java
//要求是已排序数组，num为检索元素
public static int bsearch(int[] arr,int num) {
        int start =0;
        int end = arr.length;
        while (start < end) {
            int mid = (start+end) >> 1;
            //中值比较
            if (num == arr[mid]) {
                return mid;
            } else if (num < arr[mid]) {
                end = mid;
            } else {
                start = mid;
            }
        }
        return -1;
    }
```

