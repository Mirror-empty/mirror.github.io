# LinkedList

**1.概率**

基于双向链表实现，使用Node存储链表节点信息。

![pSpShp6.png](https://s1.ax1x.com/2022/12/29/pSpShp6.png)

```
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```

每个链表存储了first和last指针：

```
transient Node<E> first;
transient Node<E> last;
```

![pSpSqAA.png](https://s1.ax1x.com/2022/12/29/pSpSqAA.png)

**2.初始化**

和ArrayList一样

```java
@Test
public void test_init() {
    // 初始化方式；普通方式
    LinkedList<String> list01 = new LinkedList<String>();
    list01.add("a");
    list01.add("b");
    list01.add("c");
    System.out.println(list01);
    
    // 初始化方式；Arrays.asList
    LinkedList<String> list02 = new LinkedList<String>(Arrays.asList("a", "b", "c"));
    System.out.println(list02);
    
    // 初始化方式；内部类
    LinkedList<String> list03 = new LinkedList<String>()\\{
        {add("a");add("b");add("c");}
    \\};
    System.out.println(list03);
    
    // 初始化方式；Collections.nCopies
    LinkedList<Integer> list04 = new LinkedList<Integer>(Collections.nCopies(10, 0));
    System.out.println(list04);
}

// 测试结果

[a, b, c]
[a, b, c]
[a, b, c]
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

Process finished with exit code 0

```


**3.遍历**



**Iterator遍历**

```java
 Iterator<Integer> iterator = list.iterator();
    while (iterator.hasNext()) {
        Integer next = iterator.next();
    }
```

**forEach循环**

```java
        list.forEach(integer -> {
            System.out.println(integer);
        });
```

**Stream流**

```java
 list.stream().forEach(integer -> {
            System.out.println(integer);
        });
```




**4.与ArrayList的比较**

ArrayList基于动态数组实现，Linked基于双向链表实现。ArrayList和LinkedList的区别可以归结为数组和链表的区别：

- 数组支持随机访问，但插入删除的代价很高，需要移动大量元素；
- 链表不支持随机访问，但插入删除只改变指针。

![pSppnuF.png](https://s1.ax1x.com/2022/12/29/pSppnuF.png)

![pSpp141.png](https://s1.ax1x.com/2022/12/29/pSpp141.png)


图为分别插入10万、100万、1000万数据在两种位置不同插入的集合。


ArrayList 头插 需要拷贝整个数组往后移。

尾插和中间只有在数据量大的时候才会提现出来。

