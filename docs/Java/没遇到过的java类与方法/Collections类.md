# Collections类

[参考中文API](https://www.apiref.com/java11-zh/java.base/java/util/Collections.html)

[官方文档](https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html)

Collections工具类提供的通用算法：

![pSeptpT.png](https://s1.ax1x.com/2023/01/09/pSeptpT.png)

### 二分查找

```java
  int binarySearch(List<? extends Comparable<? super T>> list, T key) {
    //   ArrayList有实现RandomAccess，但是LinkedList没有实现这个接口    元素数量阈值的效验 为500
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        // 小于阈值就用二分折半查找
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }

    private static <T>
    int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
        int low = 0;
        int high = list.size()-1;

        while (low <= high) {
            //  >>>1 除以2 向下取整
            int mid = (low + high) >>> 1;
            Comparable<? super T> midVal = list.get(mid);
            int cmp = midVal.compareTo(key);

            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found
    }
```

### 洗牌算法

洗牌算法，其实就是将 List 集合中的元素进行打乱，一般可以用在抽奖、摇号、洗牌等各个场景中。

```java
    public static void shuffle(List<?> list, Random rnd) {
        int size = list.size();
        if (size < SHUFFLE_THRESHOLD || list instanceof RandomAccess) {
            for (int i=size; i>1; i--)
                swap(list, i-1, rnd.nextInt(i));
        } else {
            Object[] arr = list.toArray();

            // Shuffle array
            for (int i=size; i>1; i--)
                swap(arr, i-1, rnd.nextInt(i));

            ListIterator it = list.listIterator();
            for (int i=0; i<arr.length; i++) {
                it.next();
                it.set(arr[i]);
            }
        }
    }

    @SuppressWarnings({"rawtypes", "unchecked"})
    public static void swap(List<?> list, int i, int j) {
        // instead of using a raw type here, it's possible to capture
        // the wildcard but it will require a call to a supplementary
        // private method
        final List l = list;
        // 元素置换 String set= l.set(j, l.get(i))
        l.set(i, l.set(j, l.get(i)));
    }
```

### 旋转算法

旋转算法，可以把ArrayList或者Linkedlist，从指定的某个位置开始，进行正旋或者逆旋操作。有点像把集合理解成圆盘，把要的元素转到自己这，其他的元素顺序跟随。

看图简单明了，怎么实现呢 循环队列？

![pSeeeyt.png](https://s1.ax1x.com/2023/01/09/pSeeeyt.png)

```java
    public static void rotate(List<?> list, int distance) {
        if (list instanceof RandomAccess || list.size() < ROTATE_THRESHOLD)
            rotate1(list, distance);
        else
            rotate2(list, distance);
    }

    private static <T> void rotate1(List<T> list, int distance) {
        int size = list.size();
        if (size == 0)
            return;
            // 获取旋转的位置
        distance = distance % size;
        if (distance < 0)
            distance += size;
        if (distance == 0)
            return;

        for (int cycleStart = 0, nMoved = 0; nMoved != size; cycleStart++) {
            T displaced = list.get(cycleStart);
            int i = cycleStart;
            do {
                i += distance;
                if (i >= size)
                    i -= size;
                    // 将0和旋转的位置替换掉，然后将替换掉的位置继续下移
                displaced = list.set(i, displaced);
                nMoved ++;
            } while (i != cycleStart);
        }
    }

```

参考小傅哥的

### 最大值最小值

```java
String min = Collections.min(Arrays.asList("1", "2", "3"));
String max = Collections.max(Arrays.asList("1", "2", "3"));

```

### 替换

```java
 List<String> list = new ArrayList<String>();
 list.add("7");
 list.add("4");
 list.add("8");
 list.add("8");
 Collections.replaceAll(list, "8", "9");
 
 // 测试结果： [7, 4, 9, 9]

```

### 连续集合判断

```java
@Test
public void test_indexOfSubList() {
    List<String> list = new ArrayList<String>();
    list.add("7");
    list.add("4");
    list.add("8");
    list.add("3");
    list.add("9");
    int idx = Collections.indexOfSubList(list, Arrays.asList("8", "3"));
    System.out.println(idx);
    
    // 测试结果：2
}

```

### synchronize

```java
List<String> list = Collections.synchronizedList(new ArrayList<String>());
Map<String, String> map = Collections.synchronizedMap(new HashMap<String, String>());

```

### 反转

       ```java
        Collections.reverse(list);
        ```