---
title: 浅谈HashMap和ConcurrentHashMap
date: 2018-08-31 19:50:50
categories: 面试
tags:
- Java
- 数据结构
---

本篇的源码基于我自己电脑上的 **jdk1.8.0_161** 。

## HashMap

- HashMap 在 jdk1.7 里是数组 + 链表，哈希冲突用拉链（头插法）解决。
- HashMap 在 jdk1.8 里是数组 + 链表/红黑树，哈希冲突用拉链（头插法）解决（链表太长就用红黑树）。


### 成员变量

```java
    /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;

    ...

    /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;

    /**
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * The number of key-value mappings contained in this map.
     */
    transient int size;

    /**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     */
    transient int modCount;

    /**
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;

    /**
     * The load factor for the hash table.
     *
     * @serial
     */
    final float loadFactor;
```

- DEFAULT_INITIAL_CAPACITY，默认 HashMap 的大小
- MAXIMUM_CAPACITY，最大的 HashMap 的大小
- DEFAULT_LOAD_FACTOR，默认的负载因子的大小
- TREEIFY_THRESHOLD，将链表升级成红黑树的 threshold
- UNTREEIFY_THRESHOLD，当 HashMap 删除元素后，红黑树退化成链表的 threshold
- MIN_TREEIFY_CAPACITY，升级成红黑树的最小容量，表示 HashMap 达到这个容量之后，一定会升级成红黑树或者 resize。（但是我没在别人的博客里看到过这个。。我只是按照注释理解了一下。。这个东西下面没用到）
- table[]，存放数据的数组，每个元素是一个内部类 Node
    ```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
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

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

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
    - hash 就是 hash 值
    - key 是存放的 key 值
    - value 是存放的 value 值
    - next 是指向下一个 Node 的引用，生成链表时用的，**如果升级成红黑树，会用 TreeNode（也是一个内部类） 代替 Node**
- size，HashMap 的 size
- modCount，表示 HashMap 的 structural modified 次数（比如，添加删除元素，或者 resize 都算，但是如果仅仅是值的改变不算）。当迭代操作或者序列化操作时，操作前后需要比较 modCount 是否相等，不相等就 Fail-Fast，抛出 ConcurrentModificationException。
- threshold，resize 的大小，等于 （capacity * load factor），达到 threshold 的时候就会扩容（会 rehash，复制数据等操作，比较消耗性能）
- loadFactor，负载因子

### putVal()

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

- 判断当前 table 是否未初始化，未初始化就初始化一波
- 根据 hash 值定位到具体的 bin，如果这个 bin 为空，直接新建一个 bin 放入这个 Node，**然后返回**
- 判断 key 是否和当前结点的 key 相等，相等就直接替换 Node（或者 key 都为 null 的时候，HashMap 允许 key 为 null，顺便再插一句，HashMap 中 null 无法计算其 hash 值，默认都是放到下标为 0 的  上）
- 如果当前结点是 TreeNode 红黑树结点，就按红黑树的方法插入（具体就不展开了）
- 如果是链表，就遍历链表，找到相同的 key 的 Node 就可以直接替换 Node，如果没找到就 newNode()（这个方法会在对应的 bin 中插入 Node）
- 统一替换 Node（即 e 不为空）
- modCount 自增
- 判断是否需要 resize

### get()

```java
/**
* Returns the value to which the specified key is mapped,
* or {@code null} if this map contains no mapping for the key.
*
* <p>More formally, if this map contains a mapping from a key
* {@code k} to a value {@code v} such that {@code (key==null ? k==null :
* key.equals(k))}, then this method returns {@code v}; otherwise
* it returns {@code null}.  (There can be at most one such mapping.)
*
* <p>A return value of {@code null} does not <i>necessarily</i>
* indicate that the map contains no mapping for the key; it's also
* possible that the map explicitly maps the key to {@code null}.
* The {@link #containsKey containsKey} operation may be used to
* distinguish these two cases.
*
* @see #put(Object, Object)
*/
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

/**
* Implements Map.get and related methods
*
* @param hash hash for key
* @param key the key
* @return the node, or null if none
*/
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
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

- 判断 table 是否未初始化，未初始化或者定位到的 bin 为空的话，直接**返回 null**
- 判断第一个 Node/TreeNode 的值是否为查询的 key，是的话直接返回（always check first node），若不匹配就下一步
- 判断为 TreeNode，查找红黑树
- 判断链表，遍历链表查找

### 并发下会形成环形链表

一个例子：

1. 刚开始，设 length 为 4，threshold 为 3，现在要扩容了
    - 0：
    - 1：5 -> 9 -> 17
    - 2：
    - 3：
2. thread1 执行扩容中，此时执行到 9 这个 Node：
    - 0：
    - 1：**9** -> 17
    - 2:
    - 3:
    - 4:
    - 5: 5
    - 6：
    - 7：
3. thread2 执行完成：
    - 0：
    - 1：17 -> 9 
    - 2:
    - 3:
    - 4:
    - 5: 5
    - 6：
    - 7：
4. thread1 继续执行，把 rehash 后的 9 插到 1 的 bin 上，头插插入
    - 0：
    - 1：9 <=> 17
    - 2:
    - 3:
    - 4:
    - 5: 5
    - 6：
    - 7：

这样 9 和 17 成为了对方的 next，形成了环。


## ConcurrentHashMap

其中的核心数据，如 value，以及链表都是 volatile 修饰的，保证了获取时的可见性。（可以防止出现环，get 时也不用加锁）

- ConcurrentHashMap 在 jdk1.7 中是由 Segment 数组（Segment 里存放 HashEntry 数组）构成的，Segment 继承自 ReentrantLock。
    - put() 
        - 尝试获取锁，失败说明和其他线程存在竞争，用 `canAndLockForPut()` 自旋获取锁
            - 尝试自旋获取锁
            - 如果重试次数达到 `MAX_SCAN_RETRIES`，改为用阻塞锁获取
        - 定位到对应的 HashEntry
        - 遍历，找到就替换
        - 没找到就插入一个 HashEntry，并判断是否需要扩容
        - 解锁
- ConcurrentHashMap 在 jdk1.8 中加了红黑树并且做了一点结构上的调整，抛弃了 Segment 分段锁，采用了 CAS(Compare And Swap) + synchronized 来保证并发的安全性。

### putVal()

```java
/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                            new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                    (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                        value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

- 计算出 hashCode
- 判断是否需要初始化
- 如果当前 key 定位的 Node 为空，尝试利用 CAS 写入，失败则自旋保证成功
- 如果 hashcode == MOVED，表示已经这个 Node 已经被移动过了（表示其他线程在进行扩容操作），helpTransfer() 是一个辅助方法
- 如果都不满足，即当前 key 存在，并且找到了，用 synchronized 锁写入数据
- 如果数量大于 TREEIFY_THRESHOLD，升级成红黑树


## 总结

大后天回学校！

## References

- [HashMap? ConcurrentHashMap? 相信看完这篇没人能难住你！](https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/)
- [jdk1.8中ConcurrentHashMap的实现原理](https://blog.csdn.net/fjse51/article/details/55260493)
