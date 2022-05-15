**重点概念 Q&A**

---

- **基本概念**
    - ThreadLocal，广泛的定义就是线程副本
    - ThreadLocal 贯穿于当前线程，注意，当前线程，所以，理论上而言，不同线程之间的 ThreadLocal 是相互隔离的
    - ThreadLocal 主要有两个方法，get() 和 set(Object o)，顾名思义，get 就是去取值，而 set 就是存储一个值
    - 那么ThreadLocal 的值存在哪里呢？存在  Thread.ThreadLocal.ThreadLocalMap 中
    - ThreadLocalMap 是一个自实现的 map，完全区别于 Utils 中的 HashMap，并且这个 map 的 key 是 ThreadLocal 本身，并且是个弱引用
- **内存泄漏（Key 被回收了，value 还在）**
    - 内存泄露这个问题是经常被问到的问题，主要原因就是 ThreadLocalMap 中对象的 key 被回收了，而 Value 还在内存当中，所以当这种状态下的 value 很多的时候，就造成了内存泄露
    - 那么 key 为什么会被回收呢，因为 key 是个弱引用，当产生GC 的时候，弱引用便会被回收掉
- **弱引用（Key 是弱引用，Entry extends WeakReference<ThreadLocal<?>>）**
    - 关于弱引用，只要产生了 GC，并且被扫描到了，那么弱引用的对象就会立马被回收掉
    - PS：关于软引用，产生 GC 的时候并不会立马去回收掉，而是当内存不够的时候，会优先进行回收软引用的对象
- **map 数据结构（ThreadLocal.ThreadLocalMap）**
    - ThreadLocalMap 是一个自己实现的 Map，最底层是一个 Entry 数组，初始容量是 16，
    - Entry extends WeakReference<ThreadLocal<?>>，
    - ThreadLocalMap 的 key 是 ThreadLocal 本身，而且是个弱引用，value 是 object 类型的，所以可以是任意值
- **map 扩容（reHash 是 2/3，reSize 是  $\frac 23 * \frac 34 = \frac 12$）**
    - map 底层数组的初始容量是 16，阈值是 2/3 * len
    - 关于扩容，存在一个很特别的设计，就是分两步，即 reHash 和 reSize
    - 在 set() 之后，会先进行启发式的清理（cleanSomeSlots()），然后再判断 entry 的数量，即 size 大于等于阈值（2/3 * len）之后就会 rehash，
    - 在 reHash 方法中，会先进行探测式的清理（expungeStaleEntries()），结束之后，再判断 entry 的数量，当 $size ≥ \frac 34   threshold$，即 $size ≥ \frac 12 len$ 的时候，才会真正地进行 reSize() 的操作
    - 扩容之后数组的长度会变成原先的 2 倍
- **map 中的 hash 算法（斐波那契数——黄金分割数，0x61c88647）**
    - int h = key.threadLocalHashCode & (len - 1);
    - 涉及到的关键点有两个吧，一个是 & (len - 1)，这个是基本操作了，在 hashMap 中就有这种用与运算取代取余运算的算法，基操了，不多说
    - 另外一个就是 key.threadLocalHashCode 中会涉及到的 斐波那契数 HASH_INCREMENT = 0x61c88647，用这个黄金分割数的目的只有一个，就是在一定程度上面使数据更加均匀
- **map 中是如何解决 hash 冲突的（开放地址法，即向右找下一个）**
    - hashMap 中解决 hash 冲突使用的是链地址法，即将一个链表或者一个树去解决 hash 冲突
    - 在 ThreadLocalMap 中采用的是开放地址法，即如果碰到有 hash 冲突，则向右查找下一个空的槽进行数据的插入，若果下一个槽也是满的，则再向右继续查找
- **过期 key 处理——探测式清理（expungeStaleEntry(int staleSlot)）**
    - 两步走，
    - 1、将 staleSlot 位置处的 value 置为 null，entry 置为 null
    - 2、从 staleSlot 开始往后查找，一直到 entry 为 null 位置，将这期间的 entry 进行判断，如果 key 为 null，则进行置空处理，如果不为空，则进行 rehash 操作，因为它前面有空的槽，所以要 rehash 一下，万一可以往前移动呢，是不是
- **过期 key 处理——启发式清理（cleanSomeSlots(int i, int n)）**
    - Heuristically scan some cells looking for stale entries.
    - 启发式清理，关键点有两个 ① while 循环中的判断条件 (n >>>= 1) != 0 ② key 为空时会进行探测式清理，并且将 n 重置为 16，
    - 16 >>>= 1 这种位运算，就是向右无符号移动一位，并且进行赋值操作，所以 n 会是 16 → 8 → 4 → 2 → 1 → 0
- **子线程如何引用父线程的 threadLocal 值（JDK 的 InheritableThreadLocal 和 阿里的 TransmittableThreadLocal）**
    - 例如多线程的场景下面，可以使用 ① ThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();  ② inheritableThreadLocal.get() 来获取父线程中 threadLocal 中存储的值
    - 但是在线程池中会有问题，因为线程池中的线程是复用的，
    - 所以阿里开源了 TransmittableThreadLocal 组件可以解决上述场景
- **threadLocal 应用场景（session、traceId 全链路跟踪）**