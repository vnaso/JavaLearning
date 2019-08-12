---
title: HashMap 和 ConcurrentHashMap
date: 2019/03/16 10:50
categories:
- Java
tags:
- Java
- HashMap
- ConcurrentHashMap
---

## HashMap 简介

HashMap 主要用来存放键值对. 基于哈希表的 Map 接口实现.

JDK8 之前 HashMap 由 数据+链表 组成, 数组是 HashMap 的主体, 链表则是主要为了解决哈希冲突而存在的. JDK8 之后, 当链表长度大于阀值(默认为8)时, 将链表转化为红黑树, 以减少搜索时间.

### 底层数据结构分析

#### JDK1.8 之前

JDK1.8 之前 HashMap 底层是 **数组和链表** 结合在一起使用, 也就是 **链表散列**. HashMap 通过 key 的 hashCode 经过 [扰动函数](#扰动函数) 处理过后得到 hash 值, 然后通过 `(n - 1) & hash` 判断当前元素存放的位置(n 为数组的长度). 如果当前位置存在元素, 就判断该元素与要存入的元素的 hash 值以及 key 是否相同, 如果相同, 就直接覆盖, 不相同, 就通过 [拉链法](#拉链法) 解决冲突.

#### 扰动函数

所谓扰动函数指的就是 HashMap 的 hash 方法. 使用 hash 方法也就是扰动函数, 是为了防止一些实现比较差的 hashCode() 方法造成的 **哈希冲突**, 也就是减少碰撞.

**JDK 1.7的 HashMap 的 hash 方法源码:**

```java
    static int hash(int h) {
        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).

        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

**JDK 1.8 HashMap 的 hash 方法源码:**

```java
      static final int hash(Object key) {
        int h;
        // key.hashCode()：返回散列值也就是hashcode
        // ^ ：按位异或
        // >>>:无符号右移，忽略符号位，空位都以0补齐
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

相比于 JDK1.8 的 hash 方法, 虽然原理不变, JDK 1.7 的 hash 方法的性能稍差一点, 因为扰动了 4 次.

#### 拉链法

"**拉链法**": 将链表和数组相结合, 也就是说创建一个链表数组, 数组中每一格都是一个链表. 若遇到哈希冲突, 则将冲突的值加到链表中即可.

![UTOOLS1552706070947.png](https://i.loli.net/2019/03/16/5c8c6a16382b0.png)

#### JDK1.8 之后

JDK1.8 在解决哈希冲突时有了较大的变化: 当链表长度大于阀值(默认为 8 )时, 将链表转化为红黑树, 以减少搜索时间.

![UTOOLS1552706315087.png](https://i.loli.net/2019/03/16/5c8c6b0967a29.png)

**HashMap类**:

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L;    
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table; 
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    // 加载因子
    final float loadFactor;
}
```

- loadFactor 加载因子

  loadFactor 加载因子用来控制数组存放数据的疏密程度, loadFactor 越趋近于 1, 那么数组中存放的数据(entry)也就越多, 也就越密, 链表的长度增加的可能性(碰撞的几率)就越高. 反之, loadFactor 越小(趋近于 0), 数组中存放的数据(entry)也就越少, 链表的长度增加的可能性(碰撞的几率)就越低.

  loadFactory 太大导致查找元素的效率低, 太小导致数组的利用效率低, 存放的数据会很分散. loadFactory 的默认值为 **0.75f** 是官方给出的一个比较好的临界值.

- threshold

  该属性是衡量数组是否需要扩增的一个标准. `threshold = capacity * loadFactor`, 当 `size >= threshold` 时, 就要考虑对数组的扩展. 

给定的默认容量为 16, 加载因子为 0.75. Map 在使用过程中不断往里面存放数据, 当数量达到了 `16 * 0.75 = 12` 时, 就要将当前的数组扩容, 而扩容涉及到 rehash, 复制数据等操作, 十分消耗性能.

**Node 节点类:**

```java
// 继承自 Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
       final int hash;// 哈希值，存放元素到hashmap中时用来与其他元素hash值比较
       final K key;//键
       V value;//值
       // 指向下一个节点
       Node<K,V> next;
       Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        // 重写hashCode()方法
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        // 重写 equals() 方法
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
}
```

### 源码分析

#### 构造方法

- HashMap()
- HashMap(int)
- HashMap(int,float)
- HashMap(Map<? extends K,? extends V>)

```java
    // 默认构造函数。
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all   other fields defaulted
     }
     
     // 包含另一个“Map”的构造函数
     public HashMap(Map<? extends K, ? extends V> m) {
         this.loadFactor = DEFAULT_LOAD_FACTOR;
         putMapEntries(m, false);//下面会分析到这个方法
     }
     
     // 指定“容量大小”的构造函数
     public HashMap(int initialCapacity) {
         this(initialCapacity, DEFAULT_LOAD_FACTOR);
     }
     
     // 指定“容量大小”和“加载因子”的构造函数
     public HashMap(int initialCapacity, float loadFactor) {
         if (initialCapacity < 0)
             throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
         if (initialCapacity > MAXIMUM_CAPACITY)
             initialCapacity = MAXIMUM_CAPACITY;
         if (loadFactor <= 0 || Float.isNaN(loadFactor))
             throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
         this.loadFactor = loadFactor;
         this.threshold = tableSizeFor(initialCapacity);
     }
```

#### putMapEntries 方法:

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // 判断table是否已经初始化
        if (table == null) { // pre-size
            // 未初始化，s为m的实际元素个数
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            // 计算得到的t大于阈值，则初始化阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // 已初始化，并且m元素个数大于阈值，进行扩容处理
        else if (s > threshold)
            resize();
        // 将m中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

#### put 方法

HashMap 只提供了 put 用于添加元素，putVal 方法供 put 方法调用, 并没有提供给用户使用.

**putVal** 方法添加元素:

- 如果定位到的数组位置没有元素, 直接插入.
- 如果定位到的数组位置有元素, 就和要插入的 key 比较. 
  - 相同: 直接覆盖.
  - 不同: 判断 p 是否是一个新的树节点.
    - 是, 就调用 `e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)` 将元素添加进入.
    - 不是, 就遍历链表插入.

![UTOOLS1552709029206.png](https://i.loli.net/2019/03/16/5c8c75a519154.png)

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0, 进行扩容
    // 第一次 resize 和后续的扩容有些不一样, 因为这次是数组从 null 初始化到默认的 16 或自定义的初始容量
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 确定元素存放在哪个桶中, 桶为空, 新生成结点放入桶中(此时, 这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 比较桶中第一个元素(数组中的结点)的hash值相等, key相等, 如果是, 取出这个节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // hash值不相等, 即key不相等; 判断是否为红黑树结点
        else if (p instanceof TreeNode)
            // 调用红黑树的插值方法放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 到这里说明数组上该位置为链表结点
            
            for (int binCount = 0; ; ++binCount) {
                // 在链表最末插入结点(Java 7 是插入到链表最前面)
                if ((e = p.next) == null) {
                    // 到达尾部, 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // TREEIFY_THRESHOLD 为 8, 所以如果结点数量(算上这次加入的为 9)达到阈值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等, 即节点是否存在
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // e != null 表示在桶中有找到key值、hash值与插入元素相等的结点
        // 下面的操作为进行 "值覆盖" 并返回旧值
        if (e != null) { 
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 由于插入新值导致 sieze 大于阈值, 则需要扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}  
```

#### get 方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
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

#### resize 方法

对 table 进行扩容, 会伴随着一次重新 hash 分配, 并且会遍历 hash 表中所有的元素, 十分耗时, 尽量避免.

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了, 就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // 对应使用 new HashMap(int initialCapacity) 初始化后, 第一次 put 的时候
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else { 
        // 对应使用 new HashMap() 初始化后, 第一次 put 的时候
        // signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    // 用新的数组大小初始化新的数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;	
    // 如果是初始化, 到这里就结束了, 返回 newTab 即可.
    if (oldTab != null) {
        // 数据迁移, 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 如果原数组位置上只有单个元素, 简单插入这个元素即可
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 红黑树节点迁移
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    // 链表迁移. 优化重 hash.
                    // 参考: https://blog.csdn.net/u012961566/article/details/72963157
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
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

## ConcurrentHashMap 简介

ConcurrentHashMap 和 HashMap 思路差不多, 但是因为支持并发操作, 所以要复杂一些.

Java8 以前, 整个 ConcurrentHashMap 由一个个 Segment 组成, Segment 意思是 "部分"或"一段", 所以也称为*分段锁*. Segment 通过继承 ReentrantLock 来进行加锁, 所以每次需要加锁的操作锁住的是一个 Segment, 这样只要保证每个 Segment 是线程安全的, 也就实现了全局的线程安全.

Java8 对 HashMap 进行了比较大的改动, 对于 ConcurrentHashMap, Java8 也引入了红黑树. 最难的在于扩容, 数据迁移操作不容易看懂.

### Java8 以前

![UTOOLS1558259654748.png](https://i.loli.net/2019/05/19/5ce127ca6634f90929.png)

![UTOOLS1558259850869.png](https://i.loli.net/2019/05/19/5ce1288d4f32395247.png)

Java8 以前的 ConcurrentHashMap 是由 Segment, HashEntry 组成, 和 HashMap 一样, 仍然是**数组+链表**.

ConcurrentHashMap 采用了分段锁技术, 其中 Segment 继承于 ReentrantLock. 不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理, 理论上 ConcurrentHashMap 支持 ConcurrencyLevel(Segment 数组数量)的线程并发. 每当一个线程占用锁访问一个 Segment 时, 不会影响到其他 Segment.

#### 源码解析

##### 核心成员变量

```java
	// table数组的默认长度，这个和HashMap是一样的
	static final int DEFAULT_INITIAL_CAPACITY = 16;
	// 加载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
	// 并发级别
    static final int DEFAULT_CONCURRENCY_LEVEL = 16;
	// 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
	// 这里可以看到DEFAULT_INITIAL_CAPACITY、DEFAULT_LOAD_FACTOR、MAXIMUM_CAPACITY，都是和HashMap相应字段的值是相同的。

	// 段组的最小长度，这里最小值为2的原因是，如果小于2的话(即为1)，就没有锁分段的意义了，就和Hashtable一样了，不能两个线程同时并发存和取数据了。
    static final int MIN_SEGMENT_TABLE_CAPACITY = 2;

	// 段组的最大长度
    static final int MAX_SEGMENTS = 1 << 16; 

    static final int RETRIES_BEFORE_LOCK = 2;
	// 段掩码
    final int segmentMask;
	// 段偏移量
    final int segmentShift;

	// Segment 数组, 存放数据时首先需要定位到具体的 Segment 中.
    final Segment<K,V>[] segments;

    transient Set<K> keySet;

    transient Set<Map.Entry<K,V>> entrySet;

    transient Collection<V> values;
```

##### Segment

```java
    static final class Segment<K,V> extends ReentrantLock implements Serializable {

        private static final long serialVersionUID = 2249069246763182397L;
        
        // 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
        transient volatile HashEntry<K,V>[] table;

        transient int count;

        transient int modCount;

        transient int threshold;

        final float loadFactor;
        
	}

```

##### 初始化

```java
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // Find power-of-two sizes best matching arguments
    int sshift = 0;
    int ssize = 1;
    // 计算并行级别 ssize，因为要保持并行级别是 2 的 n 次方
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    // 我们这里先不要那么烧脑，用默认值，concurrencyLevel 为 16，sshift 为 4
    // 那么计算出 segmentShift 为 28，segmentMask 为 15，后面在定位 Segment 时的散列算法中会用到这两个值
    this.segmentShift = 32 - sshift;
    this.segmentMask = ssize - 1;
 
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
 
    // initialCapacity 是设置整个 map 初始的大小，
    // 这里根据 initialCapacity 计算 Segment 数组中每个位置可以分到的大小
    // 如 initialCapacity 为 64，那么每个 Segment 或称之为"槽"可以分到 4 个
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    // 默认 MIN_SEGMENT_TABLE_CAPACITY 是 2，这个值也是有讲究的，因为这样的话，对于具体的槽上，
    // 插入一个元素不至于扩容，插入第二个的时候才会扩容
    int cap = MIN_SEGMENT_TABLE_CAPACITY; 
    while (cap < c)
        cap <<= 1;
 
    // 创建 Segment 数组，
    // 并创建数组的第一个元素 segment[0]
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    // 往数组写入 segment[0]
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```

如果使用 `new ConcurrentHashMap()` 初始化, 那么:

- Segment 数组长度为 16, 不可以扩容.
- segment[i] 的默认大小为 2, 负载因子是 0.75, 得出初始阈值为 1.5, 所以在插入第一个元素时不会触发扩容.
- 这里初始化了 segment[0], 其他位置还是 null.
- 当前 segmentShift 的值为 32 - 4 = 28, segmentMask 的值为 16 - 1 = 15.

##### put 方法

**ConcurrentHashMap 的 put 方法**

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    // 1. 计算 key 的 hash 值
    int hash = hash(key);
    // 2. 根据 hash 值找到 Segment 数组中的位置 j
    //    hash 是 32 位，无符号右移 segmentShift(28) 位，剩下低 4 位，
    //    然后和 segmentMask(15) 做一次与操作，也就是说 j 是 hash 值的最后 4 位，也就是槽的数组下标(范围就在 0~size 之间).
    int j = (hash >>> segmentShift) & segmentMask;
    // 刚刚说了，初始化的时候初始化了 segment[0]，但是其他位置还是 null，
    // ensureSegment(j) 对 segment[j] 进行初始化
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        s = ensureSegment(j);
    // 3. 插入新值到 槽 s 中
    return s.put(key, hash, value, false);
}
```

**Segment 内部(数据+链表组成)的 put 方法**

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 在往该 segment 写入前，需要先获取该 segment 的独占锁
    //    先看主流程，后面还会具体介绍这部分内容
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        // 这个是 segment 内部的数组
        HashEntry<K,V>[] tab = table;
        // 再利用 hash 值，求应该放置的数组下标
        int index = (tab.length - 1) & hash;
        // first 是数组该位置处的链表的表头
        HashEntry<K,V> first = entryAt(tab, index);
 
        // 下面这串 for 循环虽然很长，不过也很好理解，想想该位置没有任何元素和已经存在一个链表这两种情况
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                // 存在一个链表
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        // 覆盖旧值
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                // 继续顺着链表走
                e = e.next;
            }
            else {
                // 没有任何元素
                // node 到底是不是 null，这个要看获取锁的过程，不过和这里都没有关系。
                // 如果不为 null，那就直接将它设置为链表表头；如果是null，初始化并设置为链表表头。
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
 
                int c = count + 1;
                // 如果超过了该 segment 的阈值，这个 segment 需要扩容
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node); // 扩容后面也会具体分析
                else
                    // 没有达到阈值，将 node 放到数组 tab 的 index 位置，
                    // 其实就是将新的节点设置成原链表的表头
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        // 解锁
        unlock();
    }
    return oldValue;
}
```

**初始化槽(Segment): ensureSegment**

ConcurrentHashMap 初始化的时候会初始化第一个槽 segment[0], 对于其他槽来说, 在插入第一个值的时候初始化.

这里需要考虑并发, 因为可能会有很多个线程同时进来初始化同一个槽 segment[j], 不过只要有一个成功即可.

```java
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        // 这里看到为什么之前要初始化 segment[0] 了，
        // 使用当前 segment[0] 处的数组长度和负载因子来初始化 segment[k]
        // 为什么要用“当前”，因为 segment[0] 可能早就扩容过了
        Segment<K,V> proto = ss[0];
        int cap = proto.table.length;
        float lf = proto.loadFactor;
        int threshold = (int)(cap * lf);
 
        // 初始化 segment[k] 内部的数组
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
            == null) { // 再次检查一遍该槽是否被其他线程初始化了。
 
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            // 使用 while 循环，内部用 CAS，当前线程成功设值或其他线程成功设值后，退出
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    return seg;
}
```

**获取写入锁: scanAndLockForPut**

在某个 segment 中 put 操作时, 首先会调用 `node = tryLock() ? null : scanAndLockForPut(key, hash, value);`, 也就是说先进行一次 `tryLock()`, 快速获取该 segment 的独占锁, 如果失败, 就进入到 `scanAndLockForPut()` 来获取锁.

```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // negative while locating node
 
    // 循环获取锁
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {
            if (e == null) {
                if (node == null) // speculatively create node
                    // 进到这里说明数组该位置的链表是空的，没有任何元素
                    // 当然，进到这里的另一个原因是 tryLock() 失败，所以该槽存在并发，不一定是该位置
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                // 顺着链表往下走
                e = e.next;
        }
        // 重试次数如果超过 MAX_SCAN_RETRIES（单核1多核64），那么不抢了，进入到阻塞队列等待锁
        //    lock() 是阻塞方法，直到获取锁后返回
        else if (++retries > MAX_SCAN_RETRIES) {
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 // 这个时候是有大问题了，那就是有新的元素进到了链表，成为了新的表头
                 //     所以这边的策略是，相当于重新走一遍这个 scanAndLockForPut 方法
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}
```

该方法有两个出口, 一个是 `tryLock()` 成功, 循环终止, 另一个是重试次数超过 `MAX_SCAN_RETRIES`, 进入 `lock()` 阻塞等待获取独占锁. 总之就是在获取该 segment 的锁后退出, 如果需要的话, 顺便实例化一下 node.

**扩容: rehash**

扩容是针对 segment 数组某个位置内部的 HashEntry[] 进行扩容.

put 的时候, 如果判断该值插入会导致该 segment 的元素个数超过阈值, 那么**先进行扩容, 再插值**.

该方法不需要考虑并发, 因为执行该方法时, 已经持有了该 segment 的独占锁.

```java
// 方法参数上的 node 是这次扩容后，需要添加到新的数组中的数据。
private void rehash(HashEntry<K,V> node) {
    HashEntry<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    // 扩容为原来的 2 倍
    int newCapacity = oldCapacity << 1;
    threshold = (int)(newCapacity * loadFactor);
    // 创建新数组
    HashEntry<K,V>[] newTable =
        (HashEntry<K,V>[]) new HashEntry[newCapacity];
    // 新的掩码，如从 16 扩容到 32，那么 sizeMask 为 31，对应二进制 ‘000...00011111’
    int sizeMask = newCapacity - 1;
 
    // 遍历原数组，老套路，将原数组位置 i 处的链表拆分到 新数组位置 i 和 i+oldCap 两个位置
    for (int i = 0; i < oldCapacity ; i++) {
        // e 是链表的第一个元素
        HashEntry<K,V> e = oldTable[i];
        if (e != null) {
            HashEntry<K,V> next = e.next;
            // 计算应该放置在新数组中的位置，
            // 假设原数组长度为 16，e 在 oldTable[3] 处，那么 idx 只可能是 3 或者是 3 + 16 = 19
            // 因为计算位置是 与运算, 扩容时 sizeMask(newCapacity-1) 的二进制多了一位1, 即 与运算 多计算一位, 结果或不变, 或增加原来的 oldCapacity.
            int idx = e.hash & sizeMask;
            if (next == null)   // 该位置处只有一个元素，那比较好办
                newTable[idx] = e;
            else { // Reuse consecutive sequence at same slot
                // e 是链表表头
                HashEntry<K,V> lastRun = e;
                // idx 是当前链表的头结点 e 的新位置
                int lastIdx = idx;
 
                // 下面这个 for 循环会找到一个 lastRun 节点，这个节点之后的所有元素是将要放到一起的
                // lastRun 节点之后的元素都会放入 idx + oldCapacity, 之前的有可能放入 idx 或 idx + oldCapacity, 在下一个循环体中处理
                for (HashEntry<K,V> last = next;
                     last != null;
                     last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                // 将 lastRun 及其之后的所有节点组成的这个链表放到 lastIdx 这个位置
                newTable[lastIdx] = lastRun;
                // 下面的操作是处理 lastRun 之前的节点，
                //    这些节点可能分配在另一个链表中，也可能分配到上面的那个链表中
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    // 将新来的 node 放到新数组中刚刚的 两个链表之一 的 头部
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

上面两个循环, 第一个循环去除也是可以工作的, 但是这个循环下来, 如果 lastRun 的后面有比较多的节点, 那么这次就是值得的. 因为我们只需要克隆 lastRun 前面的节点, 就省去了移动之后的节点的开销. 根据 Doug Lea 所说, 根据统计, 如果使用默认的阈值, 大约只有 1/6 的节点需要克隆.

##### get 方法

1. 计算 hash 值, 找到 segment 数组中的具体位置.
2. segment 中也是一个数组, 根据 hash 找到数组(HashEntry[])中具体的位置.
3. 找到所在的链表, 顺着链表查找即可.

```java
public V get(Object key) {
    Segment<K,V> s; // manually integrate access methods to reduce overhead
    HashEntry<K,V>[] tab;
    // 1. hash 值
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    // 2. 根据 hash 找到对应的 segment
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        // 3. 找到segment 内部数组相应位置的链表，遍历
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```

#### 并发问题分析

get 过程中是没有加锁的, 需要考虑 get 的时候在同一个 segment 中发生了 put 或 remove 操作.

##### put 操作的线程安全性

1. 初始化 segment, 使用了 CAS 来初始化 Segment 中的数组(`UNSAFE.compareAndSwapObject(ss, u, null, seg = s)`).
2. 添加节点到链表的操作是插入到表头的, 所以如果这个时候 get 操作在链表遍历的过程中已经到了中间, 时不会影响的. 如果 get 在 put 之后, 需要保证刚刚插入表头的节点被读取, 这个依赖于 `setEntryAt()` 方法中使用的 `UNSAFE.putOrderedObject`.
3. 扩容. 扩容是创建了数组, 如果 get 先行, 那么就是在旧的 table 上做查询操作; 如果 put 先行, 那么 put 操作的可见性保证就是 table 使用了 volatile 关键字.

##### remove 操作的线程安全性

get 操作需要遍历链表, 但是 remove 操作会"破坏"链表. 如果 remove 破坏的节点 get 操作已经过去了, 那么不存在任何问题.

如果 remove 先行破坏了一个节点, 分两种情况考虑:

1. 如果此节点是头结点, 那么需要将头结点的 next 设置为数组该位置的元素, table 虽然使用了 volatile 修饰, 但是 volatile 并不能提供数组内部操作的可见性保证, 所以源码中使用了 UNSAGE 来操作数组, 见 `setEntryAt()`.
2. 如果要删除的节点不是头结点, 它会将删除节点的后继节点接到前驱节点中, 这里的并发保证是 next 属性是 volatile 的.

### Java8 以后

![UTOOLS1558271196547.png](https://i.loli.net/2019/05/19/5ce154e06131121711.png)

结构上和 Java8 的 HashMap 基本一样, 不过在保证线程安全性的实现比较复杂. 抛弃了原来的 Segment 分段锁, 而采用了 `CAS + synchronized`. 也将存放数据的 HashEntry 改为 Node, 但作用都是相同的. 其中  `val` 和 `next` 都用了 volatile 修饰来保证可见性.

![UTOOLS1558271420864.png](https://i.loli.net/2019/05/19/5ce155c00a53676727.png)

#### 源码解析

##### 初始化

```java
// 无参构造函数里，什么都不干
public ConcurrentHashMap() {
}
public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}
```

通过提供初始容量, 计算了 sizeCtl, sizeCtl = [(1.5 * initialCapacity + 1), 然后向上取最接近的 2 的 n 次方]. 如 initialCapacity 为 10, 那么得到的 sizeCtl 为 16; 如果 initialCapacity 为 11, 那么 sizeCtl 为 32.

##### put 方法

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    // 得到 hash 值
    int hash = spread(key.hashCode());
    // 用于记录相应链表的长度
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        // 如果数组"空"，进行数组初始化
        if (tab == null || (n = tab.length) == 0)
            // 初始化数组，后面会详细介绍
            tab = initTable();
 
        // 找该 hash 值对应的数组下标，得到第一个节点 f
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 如果数组该位置为空，
            //    用一次 CAS 操作将这个新值放入其中即可，这个 put 操作差不多就结束了，可以拉到最后面了
            //          如果 CAS 失败，那就是有并发操作，进到下一个循环就好了
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        // hash 居然可以等于 MOVED，这个需要到后面才能看明白，不过从名字上也能猜到，肯定是因为在扩容
        else if ((fh = f.hash) == MOVED)
            // 帮助数据迁移，这个等到看完数据迁移部分的介绍后，再理解这个就很简单了
            tab = helpTransfer(tab, f);
 
        else { // 到这里就是说，f 是该位置的头结点，而且不为空
 
            V oldVal = null;
            // 获取数组该位置的头结点的监视器锁
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) { // 头结点的 hash 值大于 0，说明是链表
                        // 用于累加，记录链表的长度
                        binCount = 1;
                        // 遍历链表
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // 如果发现了"相等"的 key，判断是否要进行值覆盖，然后也就可以 break 了
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            // 到了链表的最末端，将这个新值放到链表的最后面
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) { // 红黑树
                        Node<K,V> p;
                        binCount = 2;
                        // 调用红黑树的插值方法插入新节点
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            // binCount != 0 说明上面在做链表操作
            if (binCount != 0) {
                // 判断是否要将链表转换为红黑树，临界值和 HashMap 一样，也是 8
                if (binCount >= TREEIFY_THRESHOLD)
                    // 这个方法和 HashMap 中稍微有一点点不同，那就是它不是一定会进行红黑树转换，
                    // 如果当前数组的长度小于 64，那么会选择进行数组扩容，而不是转换为红黑树
                    //    具体源码我们就不看了，扩容部分后面说
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    // 对 table 长度加 1, 并检查是否需要扩容, 或者正在扩容. 如果需要扩容, 就调用扩容方法, 如果正在扩容, 就帮助其扩容.
    addCount(1L, binCount);
    return null;
}
```

**初始化数组: initTable**

主要就是初始化一个合适大小的数组, 然后设置 sizeCtl. 初始化方法中的并发问题是通过对 sizeCtl 进行一个 CAS 操作来控制.

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        // 初始化的"功劳"被其他线程"抢去"了
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        // CAS 一下，将 sizeCtl 设置为 -1，代表抢到了锁
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    // DEFAULT_CAPACITY 默认初始容量是 16
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    // 初始化数组，长度为 16 或初始化时提供的长度
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    // 将这个数组赋值给 table，table 是 volatile 的
                    table = tab = nt;
                    // 如果 n 为 16 的话，那么这里 sc = 12
                    // 其实就是 0.75 * n
                    sc = n - (n >>> 2);
                }
            } finally {
                // 设置 sizeCtl 为 sc，我们就当是 12 吧
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

##### 链表转红黑树: treeifyBin

treeifyBin 不一定会进行红黑树转换, 也可能仅仅做数组扩容.

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        // MIN_TREEIFY_CAPACITY 为 64
        // 所以，如果数组长度小于 64 的时候，其实也就是 32 或者 16 或者更小的时候，会进行数组扩容
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            // 后面我们再详细分析这个方法
            tryPresize(n << 1);
        // b 是头结点
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            // 加锁
            synchronized (b) {
 
                if (tabAt(tab, index) == b) {
                    // 下面就是遍历链表，建立一颗红黑树
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    // 将红黑树设置到数组相应位置中
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```

##### 扩容: tryPresize

这个方法要完全看懂需要看之后的 transfer 方法.

```java
// 首先要说明的是，方法参数 size 传进来的时候就已经翻了倍了
private final void tryPresize(int size) {
    // c：size 的 1.5 倍，再加 1，再往上取最近的 2 的 n 次方。
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
        tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;
 
        // 这个 if 分支和之前说的初始化数组的代码基本上是一样的，在这里，我们可以不用管这块代码
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2); // 0.75 * n
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
            break;
        else if (tab == table) {
            // 我没看懂 rs 的真正含义是什么，不过也关系不大
            // 根据容量 n 得到本次扩容唯一标识 - 摘自 https://www.cnblogs.com/stateis0/p/9062088.html
            // 参考: https://segmentfault.com/a/1190000016124883
            int rs = resizeStamp(n);

            if (sc < 0) {
                Node<K,V>[] nt;
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
            // 如果 sc 的低 16 位不等于 标识符（校验异常 sizeCtl 变化了）
            // 如果 sc == 标识符 + 1 （扩容结束了，不再有线程进行扩容）（默认第一个线程设置 sc ==rs 左移 16 位 + 2，当第一个线程结束扩容了，就会将 sc 减一。这个时候，sc 就等于 rs + 1）
            // 如果 sc == 标识符 + 65535（帮助线程数已经达到最大）
            // 如果 nextTable == null（结束扩容了）
            // 如果 transferIndex <= 0 (转移状态变化了)
            // 结束循环 
                    break;
                // 2. 用 CAS 将 sizeCtl 加 1，然后执行 transfer 方法
                //    此时 nextTab 不为 null
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            // 1. 将 sizeCtl 设置为 (rs << RESIZE_STAMP_SHIFT) + 2)
            //     我是没看懂这个值真正的意义是什么？不过可以计算出来的是，结果是一个比较大的负数
            // 如果不在扩容，将 sc 更新：标识符左移 16 位 然后 + 2. 也就是变成一个负数。高 16 位是标识符，低 16 位初始是 2.
            //  调用 transfer 方法，此时 nextTab 参数为 null
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                // 更新 sizeCtl 为负数后，开始扩容。
                transfer(tab, null);
        }
    }
}
```

这个方法的核心在于 sizeCtl 值的操作, 首先将其设置为一个负数, 然后执行 `transfer(tab, null)`, 在下一个循环将 sizeCtl 加 1, 并执行 `transfer(tab, nt)`, 之后可能是继续 sizeCtl 加 1, 并执行 `transfer(tab, nt)`.

所以, 可能的操作就是执行 1 次 `transfer(tab, null)` + n 次 `transfer(tab, nt)`. 本次扩容第一次执行时, nt 为 null, 单线程扩容, 后续在扩容过程中有其他线程进入扩容方法时, nt 不为 null, 多线程协助其扩容.

**数据迁移: transfer**

将原来的 tab 数组元素迁移到新的 nextTab 数组中.



此方法支持多线程执行, 外围调用此方法的时候, 会保证第一个发起数据迁移的线程, nextTab 参数为 null. 

原数组长度为 n, 所以我们有 n 个迁移任务, 让每个线程每次负责一个小任务是最简单的, 每做完一个任务再检测是否有其他没做完的任务, 帮助迁移就可以了. 而 Doug Lea 使用了一个 stride, 简单理解就是步长, 每个线程每次负责迁移其中的一部分, 如每次迁移 16 个小任务. 所以就需要一个全局的调度者来安排哪个线程执行哪几个任务, 这个就是属性 transferIndex 的作用.

第一个发起数据迁移的线程会将 transferIndex 指向原数组最后的位置, 然后从后往前的 stride 个任务属于第一个线程, 然后将 transferIndex 指向新的位置, 再往前的 stride 个任务属于第二个线程, 以此类推. 即将一个大的迁移任务分为了一个个子任务. transfer 这个方法并没有实现所有的迁移任务, 每次调用这个方法只实现了 transferIndex 往前 stride 个位置的迁移工作, 其他的需要外围的来控制.

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
 
    // stride 在单核下直接等于 n，多核模式下为 (n>>>3)/NCPU，最小值是 16
    // stride 可以理解为”步长“，有 n 个位置是需要进行迁移的，
    //   将这 n 个任务分为多个任务包，每个任务包有 stride 个任务
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
 
    // 如果 nextTab 为 null，先进行一次初始化
    //    前面我们说了，外围会保证第一个发起迁移的线程调用此方法时，参数 nextTab 为 null
    //       之后参与迁移的线程调用此方法时，nextTab 不会为 null
    if (nextTab == null) {
        try {
            // 容量翻倍
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        // nextTable 是 ConcurrentHashMap 中的属性
        nextTable = nextTab;
        // transferIndex 也是 ConcurrentHashMap 的属性，用于控制迁移的位置
        transferIndex = n;
    }
 
    int nextn = nextTab.length;
 
    // ForwardingNode 翻译过来就是正在被迁移的 Node
    // 这个构造方法会生成一个Node，key、value 和 next 都为 null，关键是 hash 为 MOVED
    // 后面我们会看到，原数组中位置 i 处的节点完成迁移工作后，
    //    就会将位置 i 处设置为这个 ForwardingNode，用来告诉其他线程该位置已经处理过了
    //    所以它其实相当于是一个标志。
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
 
 
    // advance 指的是做完了一个位置的迁移工作，可以准备做下一个位置的了
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
 
    /*
     * 下面这个 for 循环，最难理解的在前面，而要看懂它们，应该先看懂后面的，然后再倒回来看
     * 
     */
 
    // i 是位置索引，bound 是边界，注意是从后往前
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
 
        // 每一次自旋前的预处理，主要是定位本轮 transfer 处理的桶区间
        // advance 为 true 表示可以进行下一个位置的迁移了
        //   简单理解结局：i == transferIndex - 1, bound == transferIndex-stride
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
 
            // 将 transferIndex 值赋给 nextIndex
            // 这里 transferIndex 一旦小于等于 0，说明原数组的所有位置都有相应的线程去处理了
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                // 看括号中的代码，nextBound 是这次迁移任务的边界，注意，是从后往前
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        // 当前是处理最后一个 transfer 任务的线程, 或出现扩容冲突.
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) {
                // 所有的迁移操作已经完成
                nextTable = null;
                // 将新的 nextTab 赋值给 table 属性，完成迁移
                table = nextTab;
                // 重新计算 sizeCtl：n 是原数组长度，所以 sizeCtl 得出的值将是新数组长度的 0.75 倍
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
 
            // 之前我们说过，sizeCtl 在迁移前会设置为 (rs << RESIZE_STAMP_SHIFT) + 2
            // 然后，每有一个线程参与迁移就会将 sizeCtl 加 1，
            // 这里使用 CAS 操作对 sizeCtl 进行减 1，代表做完了属于自己的任务
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                // 任务结束，方法退出
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
 
                // 到这里，说明 (sc - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT，
                // 也就是说，所有的迁移任务都做完了，也就会进入到上面的 if(finishing){} 分支了
                finishing = advance = true;
                // 最后完成数据迁移的线程重新检查一次旧 table 中的所有桶, 看是否都被正确迁移到新 table了
                // 正常情况下, 重新检查时, 旧 table 所有桶都应该是 ForwardingNode;
                // 特殊情况下, 比如扩容冲突(多个线程申请到同一个 transfer 任务), 此时当前线程领取的任务会作废, 那么最后检查时还要处理因为作废而没有被迁移的桶, 把他们正确迁移到新 table 中.
                i = n; // recheck before commit
            }
        }
        // 如果位置 i 处是空的，没有任何节点，那么放入刚刚初始化的 ForwardingNode ”空节点“
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        // 该位置处是一个 ForwardingNode，代表该位置已经迁移过了
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            // 对数组该位置处的结点加锁，开始处理数组该位置处的迁移工作
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    // 头结点的 hash 大于 0，说明是链表的 Node 节点
                    if (fh >= 0) {
                         /**
                         * 下面的过程会将旧桶中的链表分成两部分：ln链和hn链
                         * ln链会插入到新table的槽i中，hn链会插入到新table的槽i+n中
                         */
                        //   找到原链表中的 lastRun，然后 lastRun 及其之后的节点是一起进行迁移的
                        //   lastRun 之前的节点需要进行克隆，然后分到两个链表中. 类似于 Java7 中 ConcurrentHashMap 的迁移.
                        int runBit = fh & n; // 由于 n 是 2 的幂次, 所以 runBit = 要么是 0(原位置), 要么高位是 1(原位置 + 原容量).
                        Node<K,V> lastRun = f; // lastRun 指向最后一个相邻 runBit 不同的节点.
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        // 其中的一个链表放在新数组的位置 i
                        setTabAt(nextTab, i, ln);
                        // 另一个链表放在新数组的位置 i+n
                        setTabAt(nextTab, i + n, hn);
                        // 将原数组该位置处设置为 fwd，代表该位置已经处理完毕，
                        //    其他线程一旦看到该位置的 hash 值为 MOVED，就不会进行迁移了
                        setTabAt(tab, i, fwd);
                        // advance 设置为 true，代表该位置已经迁移完毕
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        // 红黑树的迁移
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        // 如果一分为二后，节点数少于 8，那么将红黑树转换回链表
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;
 
                        // 将 ln 放置在新数组的位置 i
                        setTabAt(nextTab, i, ln);
                        // 将 hn 放置在新数组的位置 i+n
                        setTabAt(nextTab, i + n, hn);
                        // 将原数组该位置处设置为 fwd，代表该位置已经处理完毕，
                        //    其他线程一旦看到该位置的 hash 值为 MOVED，就不会进行迁移了
                        setTabAt(tab, i, fwd);
                        // advance 设置为 true，代表该位置已经迁移完毕
                        advance = true;
                    }
                }
            }
        }
    }
}
```



##### get 方法

1. 计算 hash 值.
2. 根据 hash 值找到数组对应位置: `(n - 1) & h`.
3. 根据该位置处节点性质进行相应查找
   - 如果该位置为 null, 那么直接返回 null.
   - 如果该位置处的节点刚好就是我们需要的, 返回该节点的值即可.
   - 如果该位置节点的 hash 值小于 0, 说明正在扩容, 或者是红黑树.
   - 如果以上 3 条都不满足, 那就是链表, 遍历比对即可.

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        // 判断头结点是否就是我们需要的节点
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        // 如果头结点的 hash 小于 0，说明 正在扩容，或者该位置是红黑树
        else if (eh < 0)
            // 参考 ForwardingNode.find(int h, Object k) 和 TreeBin.find(int h, Object k)
            return (p = e.find(h, key)) != null ? p.val : null;
 
        // 遍历链表
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

## 参考资料

- JavaGuide: <https://github.com/Snailclimb/JavaGuide>
- Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析: <http://www.importnew.com/28263.html>
- HashMap? ConcurrentHashMap? 相信看完这篇没人能难住你！: <https://juejin.im/post/5b551e8df265da0f84562403#heading-0>