HashMap 这个数据结构，不管是日常开发，还是求职面试，它始终都是所有 Java 程序员绕不开的宿命，所以还是决定写篇文章来详细剖析下 HashMap 这个数据结构，探探期间到底有多少奥秘。

---

## 背景

很早的时候就想写点关于数据结构方面的文章，时隔多年，终于决定正式开始提笔了，那就先从最热门的 HashMap 开始吧。

HashMap 是 Java 程序中使用率最高的数据结构之一，其主要用于处理键值对这种数据结构。而且在 JDK 1.8 中对底层的实现进行了优化，比如引入了红黑树、优化了扩容机制等。

本文主要是基于 JDK 最常用的 1.8 版本来介绍，详细分析了几个最重要的参数和方法，比如索引的计算、数组的扩容、put() 方法等，末尾也会稍微对比下 1.8 和 1.7 版本之间的差异。

## 简介

#### 特点

HashMap 继承自 Map，有如下特点：

1. 存储 key - value 类型结构，数据类型不限制  
2. 根据 key 的 hashcode 值进行存储数据  
3. 最多只允许一条记录的键(key)为 null（对 value 值不约束）  
4. 它是无序的（其实一见 hash 我们便知道了）  
5. 查询效率很高  
6. 它是线程不安全的（要线程安全，可以使用 Collections 的 synchronizedMap，或者使用更加推荐的 ConcurrentHashMap） 

它还有一些常见的兄弟姐妹，比如 LinkedHashMap、TreeMap、Hashtable，本文就不进行对比介绍了。

#### 基本结构

HashMap 的结构，是数组+链表+红黑树的结构，草图可以见下图。

> 红黑树是在 JDK1.8 版本才引入的，目的是加快链表的查询效率

![HashMap 数据结构草图](http://blogsource.chenkaikai.com/uploads/2020/03/HashMap07.png)

从上图可看出，HashMap 底层是一个哈希桶数组，名为 table，数组内存储的是基于 Node 类型的数据，所以，这个 **Node** 甚为关键，下文会详解。

然后同一个数组所以的位置可以存储多个 Node，并以链表或红黑树的形式来实现，所以很容易猜到，既然是链表，那么每个 Node 必然会记录下一个 Node。但是如果链表很长，那查询效率便会降低，所以自 JDK1.8 开始便引入了红黑树，即当链表长度超过 8 的时候，链表便会转为红黑树，另外，当链表长度小于 6 的时候，会从红黑树转为链表。

#### 链地址法

HashMap 是使用哈希表来存储数据的。哈希表为了解决冲突，一般有两种方案：**开放地址法** 和 **链地址法**。

> 开放地址法：哈希完后如果有冲突，则按照某种规则找到空位插入

HashMap 采用的便是 **链地址法**，即在数组的每个索引处都是一个链表结构，这样就可以有效解决 hash 冲突。

> 当两个 key 的 hash 值相同时，则会将他们至于数组的同一个位置处，并以链表的形式呈现。

但是如果大部分的数据都集中在了数组中的同一索引处，而其余索引处的数据比较少，即分配不均衡，则说明哈希碰撞较多。

所以为了提高 HashMap 的存取效率，我们需要将数据尽可能多地分散到数组中去，即减少哈希碰撞，为了达到这一目的，最直接的方案便是改善 hash 算法，其次是扩大哈希桶数组的大小（扩容），在下文会详细介绍。



## 字段和属性


#### 一些默认参数

``` Java
// 默认的初始容量为 16 （PS：aka 应该是 as know as）
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

// 最大容量（容量不够时需要扩容）
static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认的负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 链表长度为 8 的时候会转为红黑树
static final int TREEIFY_THRESHOLD = 8;

// 长度为 6 的时候会从红黑树转为链表
static final int UNTREEIFY_THRESHOLD = 6;

// 只有桶内数据量大于 64 的时候才会允许转红黑树
static final int MIN_TREEIFY_CAPACITY = 64;
```
初始容量是 16，可以扩容，但是扩容之后的容量，也是 2 的幂次方，比如 32、64，why？这里面涉及到很多巧妙的设计，下文介绍 resize() 方法的时候会详细介绍。

另外，我们解释下 **MIN_TREEIFY_CAPACITY**，虽然说当链表长度大于 8 的时候，链表会转为红黑树，但是也是需要满足桶内存储的数据量大于上述这个参数的值，否则不仅不会转红黑树，反而会进行扩容操作。

比如下面这段代码是判断是否要将链表转为红黑树，乍看，只是将链表长度和 **UNTREEIFY_THRESHOLD** 进行对比，其实不然，点开 `treeifyBin(tab, hash)` 这个方法，我们便可以看到，如果此时桶数组内的数据量小于 **MIN_TREEIFY_CAPACITY**，则不会将链表转红黑树，而是进行扩容操作，见下图：

![链表转红黑树](http://blogsource.chenkaikai.com/uploads/2020/03/HashMap08.png)

#### 一些重要的字段

``` Java
// Map 中存储的数据量，即 key-value 键值对的数量
transient int size;

// HashMap 内部结构发生变化的次数，即新增、删除数据的时候都会记录，
// 注意：修改某个 key 的值，并不会改变这个 modCount
transient int modCount;

// 重点，代表最多能容纳的数据量
// 即最多能容纳的 key-value 键值对的数量
int threshold;

// 负载因子，默认为 0.75
// 注意，这个值是可以大于 1 的
final float loadFactor;
```

其中有两个参数需要注意一下，一个是 **threshold**，还有一个是 **loadFactor**。

**threshold** 代表最多能容纳的 Node 数量，一般 `threshold = length * loadFactor`，也就是说要想 HashMap 能够存储更多的数据（即获得较大的 threshold），有两种方案，一种是扩容（即增大数组长度 length），另一种便是增大负载因子。

> threshold 和数组长度不是一回事哦

0.75 这个默认的负载因子的值是基于时间和空间考虑而得的一个比较平衡的点，所以负载因子我们一般不去调整，除非有特殊的需求：

1. 比如 **以空间换时间**，意思是如果内存比较大，并且需要有较高的存取效率，则可以适当降低负载因子，这样做的话，就会减小哈希碰撞的概率。  
2. 再比如 **以时间换空间**，意思是如果内存比较小，并且接受适当减小存取效率，则可以适当调大负载因子，哪怕大于 1，这样做的话，就会增大哈希碰撞的概率。

> 关于 0.75 这个负载因子的详细的解释，需要建立数学模型来分析，由于鄙人才疏学浅，暂不进行讨论。

#### Node<K,V>

HashMap 底层是一个 Node[] table，所以 Node 是一个很重要的数据结构。

Node 实现了 Entry 接口，所以，Node 本质上就是一个 Key-Value 数据结构。

``` Java
static class Node<K,V> implements Map.Entry<K,V> {
    // key 的 hash 值
    final int hash;
    final K key;
    V value;
    // 记录下一个 Node
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {...}
    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }
    public final int hashCode() {...}
    public final V setValue(V newValue) {...}
    public final boolean equals(Object o) {...}
}
```

## 构造函数

总共有四个构造函数，依次来讲解下。

![构造函数列表](http://blogsource.chenkaikai.com/uploads/2020/03/HashMap02.png)

#### HashMap()

构造一个空的 HashMap，初始容量为 16，负载因子为 0.75

``` Java
// 构造一个空的 HashMap，初始容量为 16，负载因子为默认值 0.75
public HashMap() {    
    this.loadFactor = DEFAULT_LOAD_FACTOR;  // all other fields defaulted
}
```


#### HashMap(int initialCapacity)

构造一个空的 HashMap，并指定初始化容量，负载因子为默认的 0.75。

构造函数内部会调用下文紧接着讲到的第三种构造函数。

``` Java
// 构造一个空的 HashMap，并指定初始化容量，负载因子采用默认的 0.75
public HashMap(int initialCapacity) {    
    // 调用另一个构造函数
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

#### HashMap(int initialCapacity, float loadFactor)

构造一个空的 HashMap，并指定初始化容量，指定负载因子。

``` Java
// 构造一个空的 HashMap，并指定初始化容量，指定负载因子
public HashMap(int initialCapacity, float loadFactor) {
    // 初始容量不为负数
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +  initialCapacity);
    // 初始容量大于最大值时，则取最大值
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 负载因子不能小于 0，并且必须是数字，否则抛异常
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
    
    // 最大容量 MAXIMUM_CAPACITY 为 1 << 30
}
```

构造函数中会进行一系列的参数判断，并且会进行初始化操作。

1. 如果初始容量小于 0，或者负载因子小于 0 或不为数字时，会抛出 `IllegalArgumentException`  异常。  
2. 如果初始容量大于最大值（2^30），则会使用最大容量。  
3. 设置 threshold，直接调用 `tableSizeFor()` 方法，该方法会返回一个大于等于指定容量的 2 的幂次方的整数，例如传入 6，则会返回 8。

> `tableSizeFor()` 方法的详细解释会放到下文来讲解。
> 
> 另外，直接将得到的值赋给 `threshold`，难道不是应该是这样的操作吗？`this.threshold = tableSizeFor(initialCapacity) * this.loadFactor;`, 其实不然，我们再看一眼源码，会发现初始化的动作放在了 put 操作中。


#### HashMap(Map<? extends K, ? extends V> m)

构造一个非空的 HashMap，保证初始化容量能够完全容下传进来的 Map，另外，负载因子使用的是默认值 0.75。

``` Java
// 构造一个非空的 HashMap，指定了默认的负载因子 0.75
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    // 将 Map 中的 key-value 赋值到新的 Map 中去
    putMapEntries(m, false);
}
```

`putMapEntries()` 方法是将传递进来的 Map 中的数据全都存入到当前的 HashMap 中去，方法的详解见下文。


## 关键方法

#### tableSizeFor(int cap)

顾名思义，初始化桶数组的大小。

该方法的作用是返回一个大于等于传入的参数的数值，并且返回值会满足以下两点：

1. 返回值是 2 的幂次方  
2. 返回值是最接近传入的参数的值

比如：传入 5，则返回 8；传入 8，则返回 8；

这个方法的设计是很令人惊叹的，十分巧妙，除了敬仰还是敬仰。

``` Java
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

2 的幂次方有个特点，就是它的字节码除了最高位是 1，其余全是 0。

比如 2，字节码为：10，最高位为 1，其余为 0

再比如16，字节码为：10000，最高位为 1，其余为 0

所以方法内使用了大量的 “或运算”和 “右移”操作，目的是保证从最高位起的每个 bit 都是 1。

1. 首行 `int n = cap - 1;` 的作用，是为了防止传入的参数本身就是一个 2 的幂次方，否则会返回两倍于参数的值；  
2. `n |= n >>> 1;` 的作用，是保证倒数第二个高位也是 1，下面的代码类似。  
3. 最后一行之前，得到的数类似 0000 1111 这种从第一个高位起全是 1，这样只要加了 1，则返回的数值必然是 2 的幂次方。


详细的计算过程详解见下图：

![tableSizeFor 过程](http://blogsource.chenkaikai.com/uploads/2020/03/HashMap01.png)

#### putMapEntries(Map<? extends K, ? extends V> m, boolean evict)

该方法是将参数 m 中的所有数据存入到当前的 HashMap 中去，比如在上文提到的第四种构造函数便调用了此方法。

此方法还是比较简单的，下文代码中都注释，其中涉及到的两个关键方法 `resize()` 和 `putVal()` 方法，作用分别为扩容和赋值，下文再详细介绍这两个方法。

``` Java
// 将参数 m 中的所有元素存入到当前的 HashMap 中去
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    // 获取 m 中的参数个数（key-value 的数量）
    int s = m.size();
    if (s > 0) {
        // 判断 table 是否被初始化过，否则初始化一遍。（PS：table 是在 put 操作的时候进行初始化的，所以如果当前的 HashMap 没有进行过 put 操作，则当前的 table 并不会被初始化）
        if (table == null) { // pre-size
            // 根据传进来的 map 的元素数量来计算当前 HashMap 需要的容量
            float ft = ((float)s / loadFactor) + 1.0F;
            // 计算而得的容量是不能大于最大容量的
            int t = ((ft < (float)MAXIMUM_CAPACITY) ? (int)ft : MAXIMUM_CAPACITY);
            // 将计算而得的容量赋值给 threshold，前提是大于当前容量（即不会减小当前的 HashMap 的容量）
            if (t > threshold)
                // 将容量转换为最近的 2 的 幂次方
                threshold = tableSizeFor(t);
        }
        // table 不为空，即已经初始化过了，
        // 如果 m 中的元素数量超过了当前 HashMap 的容量，则要进行扩容
        else if (s > threshold)
            resize();
        // 遍历 m 的每个元素，将它的 key-value 插入到当前的 HashMap 中去
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            // 插入数据（注意，为什么不是 put() 呢，因为 put() 其实也是调用的 putVal() 方法）
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```


#### hash(Object key)

HashMap 中的 hash 算法，是将 key 进行 hashCode 计算得到 h，然后将 h 的高16位与低16位进行异或运算。

这样做是从速度、质量等多方面综合考虑的，而且将高位和低位进行混合运算，这样是可以有效降低冲突概率的。

另外，高位是可以保证不变的，变的是低位，并且低位中掺杂了高位的信息，最后生成的 hash 值的随机性会增大。

下图举例介绍异或计算（例如 h 为 467,926,597）：

![异或运算](http://blogsource.chenkaikai.com/uploads/2020/03/HashMap05.png)

从上图中也看出，高位的数字是不变的。

``` Java
// (h = key.hashCode()) ^ (h >>> 16);
static final int hash(Object key) {
    int h;
    // 高 16 位与低 16 位进行异或运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

另外，hash() 的作用是根据 key 计算一个 hash 数值，然后根据这个 hash 数值计算得到数组的索引 i，基于这个索引我们才能进行相关的增删改查操作，所以这个索引甚是关键。

计算公式为：`i = hash(key) & (n-1)`，即下面这个方法，但是这个下面这个方法仅限 1.8 以前的版本：

``` Java
// jdk1.7 的源码，jdk1.8 没有这个方法，但是原理一样
static int indexFor(int h, int length) {  
    // 取模运算
    return h & (length-1);
}
```

这里又是一个很精妙的设计，一般情况下，我们要获取索引 i，最常用的计算方式是取模运算：`hash % length`，但是此处却使用的是：`hash & （length-1）`，妙哉妙哉。

为什么要这么做呢？因为 '%' 操作相对于位运算是比较消耗性能的，所以采用了奇淫技巧 '&' 运算。但是为什么结果是和取模运算是一致的呢？其实还是因为table的 length 的问题。

我们上文提到过，HashMap 的长度 length 始终是 2 的幂次方，这个是关键，所以才会有这种结果，简单分析见下图：

> 使用 & 位运算替代常规的 % 取模运算，性能上提高了很多，这个是 table 如此设计数组长度的优势之一，另一个很大的优势是在扩容的时候，下文会分析。

![取模运算](http://blogsource.chenkaikai.com/uploads/2020/03/HashMap09.png)


#### resize()

resize() 是一个很重要的方法，作用是扩容，从而使得 HashMap 可以存储更多的数据。

因为当我们不断向 HashMap 中添加数据时，它总会超过允许存储的数据量上限，所以必然会经历 **扩容** 这一步操作，但是 HashMap 底层是一个数组，我们都知道数组是无法增大容量的，所以 resize 的过程其实就是新建一个更大容量的数组来存储当前 HashMap 中的数据。

resize() 方法是很精妙的，我们就一起来看下 JDK1.8 的源码吧。

``` Java
final Node<K,V>[] resize() {
    // 当前 table
    Node<K,V>[] oldTab = table;
    // 当前的 table 的大小
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 当前 table 的 threshold，即允许存储的数据量阀值
    int oldThr = threshold;
    // 新的 table 的大小和阀值暂时初始化为 0
    int newCap, newThr = 0;
    // ① 开始计算新的 table 的大小和阀值
    // a、当前 table 的大小大于 0，则意味着当前的 table 肯定是有数据的
    if (oldCap > 0) {
        // 当前 table 的大小已经到了上线了，还咋扩容，自个儿继续哈希碰撞去吧 
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 新的 table 的大小直接翻倍，阀值也直接翻倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // b、当前的 table 中无数据，但是阀值不为零，说明初始化的时候指定过容量或者阀值，但是没有被 put 过数据，因为在上文中有提到过，此时的阀值就是数组的大小，所以直接把当前的阀值当做新 table 的数组大小即可
    // 回忆一下：threshold = tableSizeFor(t);
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // c、这种情况就代表当前的 table 是调用的空参构造来初始化的，所有的数据都是默认值，所以新的 table 也只要使用默认值即可
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 如果新的阀值是 0，那么就简单计算一遍就行了
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // ② 初始化新的 table
    // 这个 newTab 就是新的 table，数组大小就是上面这一堆逻辑所计算出来的
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 遍历当前 table，处理每个下标处的 bucket，将其处理到新的 table 中去
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                // 释放当前 table 数组的对象引用（for循环后，当前 table 数组不再引用任何对象）
                oldTab[j] = null;
                // a、只有一个 Node，则直接 rehash 赋值即可
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // b、当前的 bucket 是红黑树，直接进行红黑树的 rehash 即可
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // c、当前的 bucket 是链表
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 遍历链表中的每个 Node，分别判断是否需要进行 rehash 操作
                    // (e.hash & oldCap) == 0 算法是精髓，充分运用了上文提到的 table 大小为 2 的幂次方这一优势，下文会细讲
                    do {
                        next = e.next;
                        // 根据 e.hash & oldCap 算法来判断节点位置是否需要变更
                        // 索引不变
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引 + oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原 bucket 位置的尾指针不为空(即还有 node )
                    if (loTail != null) {
                        // 链表末尾必须置为 null
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        // 链表末尾必须置为 null
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

好了，下面我们就介绍下上面的源码中提到的计算索引的操作，即判断 `if ((e.hash & oldCap) == 0) `。

下图展示的是扩容前和扩容后的计算索引的方法，主要关注红色框中的内容。这种方案是没问题的，但是之前我们提到，table 的大小为 2 的幂次方，这什么要这么设计呢，期间的又一个奥秘便体现在此，请看图中的备注。

![扩容索引计算1](http://blogsource.chenkaikai.com/uploads/2020/03/HashMap10.png)

既然扩容后每个 key 的新索引的生成规则是固定有规律的，即只有两种形式，要么不变 i，要么增加原先的数组大小的量（i+n），所以我们其实并不需要真的去计算每个 key 的索引，而只需要判断索引是否不变即可。所以此处巧妙地使用了 `(e.hash & oldCap) == 0` 这个判断，着实精妙，计算的细节过程看下面的图即可。

![扩容索引计算2](http://blogsource.chenkaikai.com/uploads/2020/03/HashMap11.png)


#### put(K key, V value)

此处介绍的是 `putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict)`，因为 put() 其实就是直接调用的 putVal()；

put() 方法是 HashMap 中最常用的方法之一，我们先大体关注下 put() 方法的流程，文字就暂不赘述了，看下图便很清晰了：

![put 流程](http://blogsource.chenkaikai.com/uploads/2020/03/HashMap06.png)

下面简单分析下源码：

``` Java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// 参数 onlyIfAbsent，true：不修改已存在的 value，false：已存在则进行修改
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // ① 如果当前 table 为空则进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 计算得到索引 i，算法在上文有提到，然后查看索引处是否有数据
    // ② 如果没有数据，则新建一个新的 Node
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 索引处有数据
    else {
        Node<K,V> e; K k;
        // ③ 索引处的第一个 Node 的  key 和参数 key 是一致的，所以直接修改 value 值即可（修改的动作放在下面）
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // ④ 索引处的 bucket 是红黑树，按照红黑树的逻辑进行插入或修改
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // ⑤ 索引处的 bucket 是链表
        else {
            // 遍历链表上面的所有 Node
            for (int binCount = 0; ; ++binCount) {
                // 索引处的 Node 为尾链
                if ((e = p.next) == null) {
                    // 直接新建一个 Node 插在尾链处
                    p.next = newNode(hash, key, value, null);
                    // 判断是否需要转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        // 链表转换为红黑树，此方法在上文中也有介绍
                        treeifyBin(tab, hash);
                    break;
                }
                // 当前 Node 的 key 值和参数 key 是一致的，即直接修改 value 值即可（修改的动作放在下面）
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 找到了相同 key 的 Node，所以进行修改 vlaue 值即可
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // 修改 value 值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            // 修改操作，直接 return 结束掉代码逻辑
            return oldValue;
        }
    }
    // 记录结构发生变化的次数
    ++modCount;
    // ⑥ 判断是否需要扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    // 新增的 Node，返回 null
    return null;
}
```

#### get(Object key)

此处介绍的是 `getNode(int hash, Object key`，因为 get() 其实就是直接调用的 getNode()；

get() 方法也是比较简单的，就是根据 key 获取 table 的索引，然后再分情况查找拥有相同 key 的 Node；

源码大致如下：

``` Java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 当前 table 不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 判断索引处的第一个 Node 的 key 值是否和参数 key 相同，相同则返回该 Node
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 索引处的第一个 Node 不是想要的，则接着查 next
        if ((e = first.next) != null) {
            // bucket 是红黑树结构
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                // bucket 是链表结构
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```



## JDK1.8 VS JDK1.7

1.7 和 1.8 之间的区别，最最主要的是如下三方面，当然，鄙人觉得这三点的变化可以算是比较成功的优化。

1. 扩容后 Node 索引的计算方式不同。上文提到，由于 table 大小的这种神奇的设计，所以扩容时计算索引的时候，1.8 中只需要简单使用 & 运算来判断是否为 0 即可，并不需要像 1.7 一样每次都是用 & 运算来计算出索引值。   
2. 1.8 中引入了红黑树结构。上文也提到了，当链表长度大于 8 的时候会转换为红黑树，但是 1.7 中是数组+链表的组合。    
3. 1.8 中采用的是尾插法，而 1.7 中采用的是头插法。比如扩容的时候，头插法会使链表上 Node 的顺序调转，而尾插法则不会，另外，头插法也会造成环形链死循环等问题，本文就不深讨了。

## 其它

HashMap 是线程不安全的，因为它是允许多线程同时操作同一个数组的，比如 put()，比如 resize()，这些都会造成数据异常甚至死循环。

所以要使用线程安全的 Map 的时候，可以使用 HashTable，但是这个不推荐，或者使用自 JDK1.8 开始引入的 Concurrent 包中的 ConcurrentHashMap，这个是比较推荐的，具体的介绍就放到下文再说吧。

参考了美团的技术博客：[传送门](https://tech.meituan.com/2016/06/24/java-hashmap.html)
