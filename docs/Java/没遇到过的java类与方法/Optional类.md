# Optional类

Optional类(java.util.Optional)是一个容器类，代表一个值存在或不存在，原来用null表示一个值不存在，现在Optional可以更好的表达这个概念。并且可以避免空指针异常。

- Optional.of(T t)：创建一个Optional实例
- Optional.empty()：创建一个空的Optional实例
- Optional.ofNullable(T t)：若t不为null,创建Optional实列，否则创建空实例
- isPresent()：判断是否包含值
- orElse(T t)：如果调用对象包含值，返回该值，否则返回t
- orElseGet(Supplier s)：如果调用对象包含值，返回该值，否则返回s获取的值
- map(Function f)：如果有值对其处理，并返回处理后的Optional，否则返回Optional.empty()
- flatMap(Function mapper)：与map类似，要求返回值必须是Optional


```java
Optional<?> message = Optional.ofNullable(record.value());
```