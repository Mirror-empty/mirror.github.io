# ArrayList

**1.概览**

因为ArrayList是基于数组实现的，所以支持快速随机访问。RandomAccess(标记接口)接口标识着该类支持快速随机访问。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

数组的默认大小为10.
```
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;
```

![zzQoge.png](https://s1.ax1x.com/2022/12/27/zzQoge.png)

**2.扩容**

添加元素使用ensureCapcityInternal()（确保容量在本身足够，扩容方法）方法来保证容量足够，如果不够时，需要使用grow()方法进行扩容，新容量的大小为``oldCapacity+(oldCapacity >> 1)``,即oldCapacity+oldCapacity/2。其中oldCapcity>>1需要取整，所以新容量大约是旧容量的1.5倍左右。（oldCapacity 为偶数就是 1.5 倍，为奇数就是 1.5 倍-0.5）


扩容操作需要调用Arrays.copyOf把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建ArrayList对象就指定大概容量大小，减少扩容操作的次数。

```java
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    } 

```

**3.删除元素**

需要调用System.arraycopy()将index+1后面的元素都复制到index位置上，该操作的时间复杂度为O(N),可以看到ArrayList删除元素的代价是非常高的。

```java
 public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

**4.序列化**

ArrayList基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。

保存元素的数组elementDate使用transient修饰,该关键字声明数组默认不会被序列化。

```java
transient Object[] elementData; // non-private to simplify nested class access
```
ArrayList实现了writeObject()和readObject()来控制只序列化数组中有元素填充那部分内容。

```java
 private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    /**
     * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,
     * deserialize it).
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```

序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为字节流并输出。而writeObject()方法在传入的对象存在writeObject()的时候会去反射调用该对象的writeObject() 来实现序列化。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。

```java
ArrayList list = new ArrayList();
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
oos.writeObject(list);
```

**5.Fail-Fast**

modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。


在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。代码参考上节序列化中的 writeObject() 方法。





### ArrayList的创建


**内部类创建**

```java
       ArrayList<String> a=new ArrayList<String>() {
            {
                add("a");
                add("b");
            }
        };
```

**Array.asList创建**

```java
ArrayList<String> a=new ArrayList<String>(Arrays.asList("a","b")); 
```
不过
- Arrays.asList 构建的集合，不能赋值给 ArrayList
- Arrays.asList 构建的集合，不能再添加元素
- Arrays.asList 构建的集合，不能再删除元素

因为Arrays.asList构建出来的List与new ArrayList得到的List，压根就不是一个List！

**Collections.ncopies创建**


返回由指定对象的n副本组成的不可变列表。 新分配的数据对象很小（它包含对数据对象的单个引用）。 此方法与List.addAll方法结合使用可以增长列表。 返回的列表是可序列化的。
参数类型
T - 要复制的对象的类和返回列表中的对象的类。
参数
n - 返回列表中的元素数。
o - 在返回的列表中重复出现的元素。
结果
一个不可变的列表，由指定对象的 n副本组成。

```java
        ArrayList<String> a=new ArrayList<String>(Collections.nCopies(10,"a"));
```
