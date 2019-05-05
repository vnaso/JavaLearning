---
title: 集合框架
date: 2019/03/16 20:14
categories:
- Java
tags:
- Java
---

## 集合框架

Java 中的集合框架分为两大类: Collection 和 Map, 两者区别在于:

1. Collection 是单列集合, Map 是双列集合
2. Collection 中只有 Set 要求元素唯一, Map 中键(key)需要唯一
3. Collection 的数据结构是针对元素的, Map 的数据结构是针对键的

### Collection

Collection 主要分为 List 和 Set

- **List**: 存取有序, 有索引, 可以根据索引取值, 元素可以重复
- **Set**: 存取无序, 元素不可以重复

#### List

- **ArrayList**: 底层数据结构是数组, 所以查询速度快, 增删速度慢. 非线程安全.
- **LinkedList**: 底层数据结构是链表, 所以查询慢, 增删快. 非线程安全.
- **Vector(已过时)**: 底层数据结构是数组, 所以查询速度快, 增删速度慢. 线程安全.

**ArrayList 和 LinkedList 的异同**

1. **是否线程安全**: ArrayList 和 LinkedList 都是不同步的, 非线程安全
2. **底层数据结构**: 
   - ArrayList 底层使用的是 Object 数组.
   - LinkedList 底层使用的是双向链表数据结构.
3. **插入和删除是否受元素位置的影响**:
   - ArrayList 采用数组存储, 所以插入和删除元素的时间复杂度受元素位置的影响.
   - LinkedList 采用链表存储, 所以插入, 删除元素时间复杂度不受元素位置影响.
4. **是否支持快速随机访问**: 
   - LinkedList 不支持高效的随机元素访问
   - ArrayList 支持, 通过元素的索引快速获取元素对象
5. **内存空间占用**:
   - ArrayList 的空间浪费主要体现在在 List 列表的结尾会预留一定的容量空间.
   - LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间(因为要存放前前驱, 后继节点以及数据)
6. **RandomAccess接口**:

```java
public interface RandomAccess{
}
```

RandomAccess 接口中什么都没有定义. 也就是说, 这个接口充当一个标识的作用, 标识实现这个接口的类具有随机访问功能.

在 `binarySearch()` 方法中, 它要判断传入的 List 是否是 RandomAccess 的实例.

- 如果是, 调用 `indexedBinarySearch()` 方法.
- 如果不是, 调用 `iteratorBinarySearch()` 方法.

```java
public static <T>
int binarySearch(List<? extends Comparable<? super T>> list, T key) {
  if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
      return Collections.indexedBinarySearch(list, key);
  else
      return Collections.iteratorBinarySearch(list, key);
}
```

ArrayList 实现了 RandomAccess 接口, 而 LinkedList 没有实现. 因为 ArrayList 底层数据结构是数组, 而 LinkedList 底层数据结构是双向链表. 它们实现随机访问的时间复杂度分别为 O(1) 和 O(n).

7. **List 遍历方式的选择**
   - 实现了 RandomAccess 接口的 List: 优先选择普通 for 循环, 其次 foreach.
   - 未实现 RandomAccess 接口的 List: 优先选择 Iterator 遍历(foreach 遍历底层实现也是通过 Iterator), size 大的数据, 不要使用普通 for 循环.

#### Set

- **HashSet**: 底层实际上是一个 HashMap 实例, 不保证迭代顺序, 非线程安全, 允许元素为 Null, 初始容量非常影响迭代性能.

  > HashMap 中 key 的值就是 HashSet 的值, 而 value 是 `final Object PRESENT = new Object()` , 操作 HashSet 元素实际上就是操作 HashMap.

- **LinkedHashSet**: 迭代有序, 允许元素为 Null, 底层实际上是一个 HashMap + 双向链表实例(即 LinkedHashMap), 非线程安全, 初始容量不影响迭代性能.

- **TreeSet**: 可以实现排序功能(添加时排序), 底层实际上是 TreeMap 实例, 非线程安全.

  > 保证 TreeSet 元素唯一性方法:
  >
  > 1. 自定义对象实现 Comparable 接口, 重写 `compareTo()` 方法.
  > 2. 创建 TreeSet 时, 向构造器中传入比较器 Comparator 接口实现类对象, 实现 Comparator 接口重写 `compare()` 方法(用匿名类方式).
  >
  > 向 TreeSet 存入自定义对象, 如果自定义类没有实现 Comparable 接口, 或者没有传入 Comparator 比较器时, 会报 ClassCastException 异常.

### Map

**特点**: 保存的是键值对, 键唯一, 值可以重复.

- **HashMap**: 底层数据结构为散列表, 无序, 允许为 Null, 非线程安全, 初始容量和装载因子对 HashMap 影响较大.

  > 具体了解: [HashMap](./HashMap.md)

- **LinkedHashMap**:存取有序, 底层数据结构为散列表和双向链表, 允许为 Null, 非线程安全, 初始容量和装载因子对 HashMap 影响较大.

- **TreeMap**: 有序, 底层数据结构是红黑树, 非线程安全, 使用 Comparator 或 Comparable 来比较是否相等以及排序. 

  > 如果 Comparator 为 Null, 则使用 Key 作为比较器进行比较, 并且 key 必须实现 Comparable 接口.

  **HashMap 和 HashTable 的区别**

  1. HashMap 是非线程安全的, HashTable 是线程安全的. HashTable 内部的方法基本都经过 synchronized 修饰.
  2. 因为线程安全的问题, HashMap 比 HashTable 效率高.
  3. HashMap 允许有 Null 存在, 而在 HashTable 中不允许 Null.

  如果想要线程安全, 可以使用 ConcurrentHashMap.

  ConcurrentHashMap 对整个桶数组进行了分割分段(Segment), 然后在每一个分段上都用 lock 锁进行保护, 相对于 HashTable 的 synchronized 锁的粒度更精细, 并发性能更好. (JDK1.8 之后 ConcurrentHashMap 启用了一种全新的方式实现, 利用 CAS 算法). ConcurrentHashMap 不允许 Null.

### Queue

- **PriorityQueue**: 不允许 Null, 按元素大小进行重新排序, 底层数据结构是最小堆, 非线程安全.

  > 在遍历时, 如果不需要删除元素, 以 peek 的方式遍历每个元素.
  >
  > `Iterator()` 中提供的迭代器并不保证以有序的方式遍历其中的元素.

- **Deque**: 是一个双端队列, 还可以当做栈使用.

  - **ArrayQueue**: 底层数据结构是数组, 是循环队列, 通过 head 和 tail 两个游标来实现. 非线程安全, 

### 集合的选用

- 需要根据键值来获取元素值: Map
  - 需要排序: TreeMap
  - 不需要排序: HashMap
  - 保证线程安全: ConcurrentHashMap
- 只需要存放元素值: Collection
  - 需要保证元素唯一: TreeSet 或 HashSet
  - 不要要元素唯一: ArrayList 或 LinkedList

## 参考资料

- Java集合总结: <https://juejin.im/post/5ad40593f265da23750759ad>
- Java集合入门和深入学习: <https://juejin.im/post/5ad82dbef265da503825b240>
- Java集合（七） Queue详解: <https://juejin.im/post/5a3763ed51882506a463b740>

