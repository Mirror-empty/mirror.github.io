# PriorityQueue优先队列

本质是一个堆，默认情况下堆顶每次都保留最小值，每插入一个元素，仍动态维护堆顶为最小值。

**初始化**

>PriorityQueue()//使用默认的初始容量（11）创建一个 PriorityQueue，并根据其自然顺序对元素进行排序。

```java
PriorityQueue<Integer> Q = new PriorityQueue<>(); // 初始化
```


**常用函数**

- add(E e)//将指定的元素插入此优先级队列。
- clear()//清空
- contains(Object o) // 如果包含指定元素返回true
- iterator()//返回在此队列中的元素上进行迭代的迭代器。
- offer(E e) // 将指定元素插入此优先队列
- peek() // 获取第一个元素，及最小或最大元素
- poll() // 获取并移除第一个
- remove(Object o) // 移除指定元素
- size() // 返回元素个数

**实现大根堆的方法**

使用自定义比较器

如何用 PriorityQueue 实现大根堆 queueA？

这里的知识点包括：

PriorityQueue 默认是小根堆，大根堆需要重写比较器。

可以在 new PriorityQueue<>() 中的参数部分加入比较器。

具体写法是：(v1, v2) -> v2 - v1。

Queue 类的输入是 offer() 方法，弹出是 poll() 方法。

Lamb表达式写法
```java
Queue<Integer> queueA = new PriorityQueue<>((v1, v2) -> v2 - v1);
queueA.offer(1);
queueA.poll();
queueA.size();
```

```java
A=new PriorityQueue<>(new Comparator<Integer>() {

			@Override
			public int compare(Integer o1, Integer o2) {
				// TODO 自动生成的方法存根
				return o2-o1;
			}
});
```
升序标准写法

```java
< return -1 //意思就是
= return 0
> return 1
```

降序
```java
<  return 1 
= return 0 
> return -1 
```

```java
PriorityQueue<Integer> pq=new PriorityQueue<>(Collections.reverseOrder());

```
