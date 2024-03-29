# Arrays类


工具类

### sort的使用

**数组从大到小**
```java
Comparator<Integer> cmp = new Comparator<Integer>() {
				public int compare(Integer i1, Integer i2) {
					return i2-i1;
				}
			};
			Arrays.sort(arr, cmp);
```

**二维数组排序**

这里排的是数组位置为1的序列，为0就o1[0]
```java
        Arrays.sort(intervals, new Comparator<int[]>() {
            @Override
            public int compare(int[] o1, int[] o2) {
                return (o1[1] < o2[1]) ? -1 : ((o1[1] == o2[1]) ? 0 : 1);
            }
        });
```

Lam表达式 o->o[1]

[参考](https://blog.csdn.net/weixin_42632962/article/details/121999562?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-121999562-blog-100176410.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-121999562-blog-100176410.pc_relevant_default&utm_relevant_index=6)


**用过的sort，binarySearch，copy**

Arrays 类是一个工具类，其中包含了数组操作的很多方法。这个 Arrays 类里均为 static 修饰的方法（static 修饰的方法可以直接通过类名调用），可以直接通过 Arrays.xxx(xxx) 的形式调用方法。

- 1）int binarySearch(type[] a, type key)
使用二分法查询 key 元素值在 a 数组中出现的索引，如果 a 数组不包含 key 元素值，则返回负数。调用该方法时要求数组中元素己经按升序排列，这样才能得到正确结果。

- 2）int binarySearch(type[] a, int fromIndex, int toIndex, type key)
这个方法与前一个方法类似，但它只搜索 a 数组中 fromIndex 到 toIndex 索引的元素。调用该方法时要求数组中元素己经按升序排列，这样才能得到正确结果。

- 3）type[] copyOf(type[] original, int length)
这个方法将会把 original 数组复制成一个新数组，其中 length 是新数组的长度。如果 length 小于 original 数组的长度，则新数组就是原数组的前面 length 个元素，如果 length 大于 original 数组的长度，则新数组的前面元索就是原数组的所有元素，后面补充 0（数值类型）、false（布尔类型）或者 null（引用类型）。

- 4）type[] copyOfRange(type[] original, int from, int to)
这个方法与前面方法相似，但这个方法只复制 original 数组的 from 索引到 to 索引的元素。

- 5）boolean equals(type[] a, type[] a2)
如果 a 数组和 a2 数组的长度相等，而且 a 数组和 a2 数组的数组元素也一一相同，该方法将返回 true。

- 6）void fill(type[] a, type val)
该方法将会把 a 数组的所有元素都赋值为 val。

- 7）void fill(type[] a, int fromIndex, int toIndex, type val)
该方法与前一个方法的作用相同，区别只是该方法仅仅将 a 数组的 fromIndex 到 toIndex 索引的数组元素赋值为 val。

- 8）void sort(type[] a)
该方法对 a 数组的数组元素进行排序。

- 9）void sort(type[] a, int fromIndex, int toIndex)
该方法与前一个方法相似，区别是该方法仅仅对 fromIndex 到 toIndex 索引的元素进行排序。

- 10）String toString(type[] a)
该方法将一个数组转换成一个字符串。该方法按顺序把多个数组元素连缀在一起，多个数组元素使用英文逗号,和空格隔开。

下面程序示范了 Arrays 类的用法。
```java
public class ArraysTest {
    public static void main(String[] args) {
        // 定义一个a数组
        int[] a = new int[] { 3, 4, 5, 6 };
        // 定义一个a2数组
        int[] a2 = new int[] { 3, 4, 5, 6 };
        // a数组和a2数组的长度相等，毎个元素依次相等，将输出true
        System.out.println("a数组和a2数组是否相等：" + Arrays.equals(a, a2));
        // 通过复制a数组，生成一个新的b数组
        int[] b = Arrays.copyOf(a, 6);
        System.out.println("a数组和b数组是否相等：" + Arrays.equals(a, b));
        // 输出b数组的元素，将输出[3, 4, 5, 6, 0, 0]
        System.out.println("b 数组的元素为：" + Arrays.toString(b));
        // 将b数组的第3个元素（包括）到第5个元素（不包括）賦值为1
        Arrays.fill(b, 2, 4, 1);
        // 输出b数组的元素，将输出[3, 4, 1, 1, 0, 0]
        System.out.println("b 数组的元素为：" + Arrays.toString(b));
        // 对b数组进行排序
        Arrays.sort(b);
        // 输出b数组的元素.将输出[0,0,1,1,3,4]
        System.out.println("b数组的元素为：" + Arrays.toString(b));
    }
}

```
Arrays 类处于 java.util 包下，为了在程序中使用 Arrays 类，必须在程序中导入 java.util.Arrays 类。

还有java8中的stream流

[参考](http://c.biancheng.net/view/5885.html)