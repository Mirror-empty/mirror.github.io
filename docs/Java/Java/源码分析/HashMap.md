# HashMap

为了便于理解，以下源码分析以 JDK 1.7 为主。

### 存储结构

内部包含了一个Entry类型的数组table。Entry存储着键值对。它包含了四个字段，从next字段我们可以看出Entry是一个链表。即数组中每个位置都被当成一个桶，一个桶存放一个链表。Hashmap使用拉链法来解决冲突，同一个链表中存放哈希值和散列桶取模运算结果相同的Entry。

![pSpuIjH.png](https://s1.ax1x.com/2022/12/29/pSpuIjH.png)

```java
transient Entry[] table;
```

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;

    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }

    public final K getKey() {
        return key;
    }

    public final V getValue() {
        return value;
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }

    public final int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    public final String toString() {
        return getKey() + "=" + getValue();
    }
}
```

### 拉链法的工作原理

```java
HashMap<String, String> map = new HashMap<>();
map.put("K1", "V1");
map.put("K2", "V2");
map.put("K3", "V3");
```
我们默认初始化的Map大小是16个长度 DEFAULT_INITIAL_CAPACITY = 1 << 4，所以获取的Hash值并不能直接作为下标使用，需要与数组长度进行取模运算得到一个下标值，也就是我们上面做的散列列子。

- 新建一个HashMap，默认大小为16；
- 插入<K1,V1>键值对，先计算K1的hashCode为115，使用除留余数法得到所在的桶下标115%16=3。
- 插入 <K2,V2> 键值对，先计算 K2 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6。
- 插入 <K3,V3> 键值对，先计算 K3 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6，插在 <K2,V2> 前面。

应该注意到链表的插入是以头插法方式进行的，例如上面的<K3,V3>不是插在<K2,V2>后面，而是擦好人在链表头部。

查找需要分成两步进行：

- 计算键值对所在的桶；
- 在链表上顺序查找，时间复杂度显示和链表的长度成正比。

![pSpuv8S.png](https://s1.ax1x.com/2022/12/29/pSpuv8S.png)

### hashCode为什么使用31作为乘数

```java
 public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

1. 31是个奇质数，如果选择偶数会导致成绩运算时数据溢出。
2. 另外在二进制中，2个5次方是32，那么也就是``31*i==(i<<5-i)``。这主要是说乘积运算可以使用位移提升性能，同时目前的JVM虚拟机也会自动支持此类的优化。
3. 使用31hash碰撞的概率更小，更稳定，散列结果更加均匀。


### 扰动函数

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

整个hashcode往右移16位，左边补0，然后在于hashcode 异或（相同为0，不同为1）然后在&数组下标（相同为1）。

    ···
增加随机，散列更加均匀，减少碰撞。

这里所有的元素存放都需要获取一个索引位置，而如果元素的位置不够散列碰撞严重，那么就失去了散列表存放的意义，没有达到预期的性能。
在获取索引ID的计算公式中，需要数组长度是2的幂次方，那么怎么进行初始化这个数组大小。
数组越小碰撞的越大，数组越大碰撞的越小，时间与空间如何取舍。
目前存放7个元素，已经有两个位置都存放了2个字符串，那么链表越来越长怎么优化。
随着元素的不断添加，数组长度不足扩容时，怎么把原有的元素，拆分到新的位置上去。
以上这些问题可以归纳为；扰动函数、初始化容量、负载因子、扩容方法以及链表和红黑树转换的使用等。接下来我们会逐个问题进行分析。

#2. 扰动函数


### 初始化容量

初始化构造方法

```java
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
```

- 阈值为threshold，通过tableSizefor方法计算，参数是初始化的值
- 这个方法也就是要寻找比初始值大的，最小的那个2进制数值。比如传了17，我应该找到的是32（2的4次幂是16<17,所以找到2的5次幂32）。

```java
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

不断的向右移就是为了用1将每个位置都填上。最后加1，把前面减的加回来。

初始化找2的次方。更好的散列。

### 负载因子

负载因子是做什么的？

负载因子，可以理解成一辆车可承重重量超过某个阈值时，把货放到新的车上。

```java

    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

当hashmap中的阈值存储到0.75时，就会发生扩容。

### [扩容链表拆分](https://bugstack.cn/md/java/interview/2020-08-07-%E9%9D%A2%E7%BB%8F%E6%89%8B%E5%86%8C%20%C2%B7%20%E7%AC%AC3%E7%AF%87%E3%80%8AHashMap%E6%A0%B8%E5%BF%83%E7%9F%A5%E8%AF%86%EF%BC%8C%E6%89%B0%E5%8A%A8%E5%87%BD%E6%95%B0%E3%80%81%E8%B4%9F%E8%BD%BD%E5%9B%A0%E5%AD%90%E3%80%81%E6%89%A9%E5%AE%B9%E9%93%BE%E8%A1%A8%E6%8B%86%E5%88%86%EF%BC%8C%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E3%80%8B.html)



### put插入

小傅哥的流程图：

![pSAXMvt.png](https://s1.ax1x.com/2023/01/06/pSAXMvt.png)


解释一些属性：


- table 为一个桶数组，在put e元素时，会根据e元素的key计算对应的下标，放入这个数组中，如果已经存放，头插
- ``DEFAULT_INITIAL_CAPACITY`` Table数组的初始化长度： 1 << 42^4=16
- ``MAXIMUM_CAPACITY`` Table数组的最大长度： 1<<302^30=1073741824
- ``DEFAULT_LOAD_FACTOR`` 负载因子：默认值为0.75。 当元素的总个数>当前数组的长度 * 负载因子。数组会进行扩容，扩容为原来的两倍
- ``TREEIFY_THRESHOLD`` 链表树化阙值： 默认值为 8 。表示在一个node（Table）节点下的值的个数大于8时候，会将链表转换成为红黑树。
- ``NTREEIFY_THRESHOLD`` 红黑树链化阙值： 默认值为 6 。 表示在进行扩容期间，单个Node节点下的红黑树节点的个数小于6时候，会将红黑树转化成为链表。
- ``MIN_TREEIFY_CAPACITY`` = 64 最小树化阈值，当Table所有元素超过改值，才会进行树化（为了防止前期阶段频繁扩容和树化过程冲突）。
![pSAvS61.png](https://s1.ax1x.com/2023/01/06/pSAvS61.png)

put插入流程

1. 首先进行哈希值的扰动，获取一个新的哈希值。(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
2. 判断tab是否为空或者长度为，如果则是进行扩容操作。
```java
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;
```
3. 根据哈希值计算下标，如果对于下标正好没有存放数据，则直接插入即可否则需要覆盖。``tab[i=(n-1)&hash]``
4. 如果去覆盖 判断tab[i]是否为树节点，否则向链表中插入数据，是则向树中插入节点。
5. 如果链表中插入节点的时候链表长达大于等于8，则需要把链表转换成红黑树。``treeifyBin(tab,hash)``
6. 最后所有元素处理完成后，判断是否超过阈值；``threshold``，超过则扩容。
7. ``treeifyBin`` 是一个链表转树的方法，但不是所有的链表长度为8后都会转成树，还需要判断存放key值的数组桶长度是否小于64``MIN_TREEIFY_CAPACXIY(table数组的初始化长度)``。如果小于则需要扩容，扩容后链表上的数据会被拆分散列的相应的桶节点上，也就把链表长度缩短了。

```java
// Node继承自Entry
  static class Node<K,V> implements Map.Entry<K,V> {

 public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 初始化桶数组 table，table 被延迟到插入新数据时再进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 如果桶中不包含键值对节点引用，则将新键值对节点的引用存入桶中即可
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 如果键的值以及节点 hash 等于链表中的第一个键值对节点时，则将 e 指向该键值对
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
            
        // 如果桶中的引用类型为 TreeNode，则调用红黑树的插入方法
        else if (p instanceof TreeNode)  
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 对链表进行遍历，并统计链表长度
            for (int binCount = 0; ; ++binCount) {
                // 链表中不包含要插入的键值对节点时，则将该节点接在链表的最后
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 如果链表长度大于或等于树化阈值，则进行树化操作
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                
                // 条件为 true，表示当前链表包含要插入的键值对，终止遍历
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        
        // 判断要插入的键值对是否存在 HashMap 中
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // onlyIfAbsent 表示是否仅在 oldValue 为 null 的情况下更新键值对的值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 键值对数量超过阈值时，则进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

```


### 扩容机制

![pSAxZUU.png](https://s1.ax1x.com/2023/01/06/pSAxZUU.png)

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // Cap 是 capacity 的缩写，容量。如果容量不为空，则说明已经初始化。
    if (oldCap > 0) {
        // 如果容量达到最大1 << 30则不再扩容
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        
        // 按旧容量和阀值的2倍计算新容量和阀值
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
    
        // initial capacity was placed in threshold 翻译过来的意思，如下；
        // 初始化时，将 threshold 的值赋值给 newCap，
        // HashMap 使用 threshold 变量暂时保存 initialCapacity 参数的值
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        // 这一部分也是，源代码中也有相应的英文注释
        // 调用无参构造方法时，数组桶数组容量为默认容量 1 << 4; aka 16
        // 阀值；是默认容量与负载因子的乘积，0.75
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    
    // newThr为0，则使用阀值公式计算容量
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    
    @SuppressWarnings({"rawtypes","unchecked"})
        // 初始化数组桶，用于存放key
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 如果旧数组桶，oldCap有值，则遍历将键值映射到新数组桶中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // 这里split，是红黑树拆分操作。在重新映射时操作的。
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 这里是链表，如果当前是按照链表存放的，则将链表节点按原顺序进行分组{这里有专门的文章介绍，如何不需要重新计算哈希值进行拆分《HashMap核心知识，扰动函数、负载因子、扩容链表拆分，深度学习》}
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    
                    // 将分组后的链表映射到桶中
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}

```

1. 扩容时计算出新的newCap、newThr，这是两个单词的缩写，一个是Capacity ，另一个是阀Threshold
2. newCap用于创新的数组桶 new Node[newCap];
3. 随着扩容后，原来那些因为哈希碰撞，存放成链表和红黑树的元素，都需要进行拆分存放到新的位置中

### 链表转红黑树

![pSAziJe.png](https://s1.ax1x.com/2023/01/06/pSAziJe.png)

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 这块就是我们上面提到的，不一定树化还可能只是扩容。主要桶数组容量是否小于64 MIN_TREEIFY_CAPACITY 
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
    	// 又是单词缩写；hd = head (头部)，tl = tile (结尾)
        TreeNode<K,V> hd = null, tl = null;
        do {
            // 将普通节点转换为树节点，但此时还不是红黑树，也就是说还不一定平衡
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            // 转红黑树操作，这里需要循环比较，染色、旋转。关于红黑树，在下一章节详细讲解
            hd.treeify(tab);
    }
}

```

1. 链表树化的条件有两点：链表长度大于等于8，桶容量大于64，才会
2. 将链表转换为树节点，此时的树可能不是一颗平衡树。同时在树转换过程中会记录链表的顺序，tl.next = p，这主要方便后续树转链表和拆分更方便。
```java
   // For treeifyBin
    TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        return new TreeNode<>(p.hash, p.key, p.value, next);
    }
```
3. 链表转换成树完成后，在进行红黑树的转换。先简单介绍下，红黑树的转换需要染色和旋转，以及比对大小。


### 查找

![pSAzDSJ.png](https://s1.ax1x.com/2023/01/06/pSAzDSJ.png)

```java
public V get(Object key) {
    Node<K,V> e;
    // 同样需要经过扰动函数计算哈希值
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 判断桶数组的是否为空和长度值
    if ((tab = table) != null && (n = tab.length) > 0 &&
        // 计算下标，哈希值与数组长度-1
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // TreeNode 节点直接调用红黑树的查找方法，时间复杂度O(logn)
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 如果是链表就依次遍历查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

```

1. 扰动函数的使用，获取新的哈希值，这在上一章节已经讲过
2. 下标的计算，同样也介绍过 tab[(n - 1) & hash])
3. 确定了桶数组下标位置，接下来就是对红黑树和链表进行查找和遍历操作了


### 删除

```java
 public V remove(Object key) {
     Node<K,V> e;
     return (e = removeNode(hash(key), key, null, false, true)) == null ?
         null : e.value;
 }
 
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    // 定位桶数组中的下标位置，index = (n - 1) & hash
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        // 如果键的值与链表第一个节点相等，则将 node 指向该节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            // 树节点，调用红黑树的查找方法，定位节点。
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                // 遍历链表，找到待删除节点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        
        // 删除节点，以及红黑树需要修复，因为删除后会破坏平衡性。链表的删除更加简单。
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
} 

```

### 遍历

1. entrySet() entrySet()返回的里面含有key,value的值，在遍历的时候只需要getKey(),getValue(),的方式来得到key，value。直接用Entry类也可以
2. keySet() 返回的是map中的key的集合，所以只需要用Set<String> 来接收即可。通过map.get(key)来得到value.


### 与Hashtable的比较

- Hashtable使用synchronize来进行同步
- HashMap可以插入键为null的Entry
- HashMap的迭代器是fail-fast迭代器
- HashMap不能保证随着时间的推移Map中的元素次序不变的。
