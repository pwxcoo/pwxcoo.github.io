---
title: 【Source Code】HashMap
date: 2020-02-25 23:03:14
categories: source
tags:
- java
- source_code
---

通过 HashMap 的 field 和 method（`hash()`, `put()`, `get()`, `resize()`）入手了解了一下 Java 中 HashMap 的设计。

## 存储结构-字段

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
- MIN_TREEIFY_CAPACITY，升级成红黑树的最小容量，表示 HashMap 达到这个容量之后，一定会升级成红黑树或者 resize。 (但是我没在别人的博客里看到过这个。。我只是按照注释理解了一下。。这个东西下面没用到)
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
    - next 是指向下一个 Node 的引用，生成链表时用的，**如果升级成红黑树，会用 TreeNode (也是一个内部类)  代替 Node**
- size，HashMap 的 size
- modCount，表示 HashMap 的 structural modified 次数 (比如，添加删除元素，或者 resize 都算，但是如果仅仅是值的改变不算) 。当迭代操作或者序列化操作时，操作前后需要比较 modCount 是否相等，不相等就 Fail-Fast，抛出 ConcurrentModificationException。
- threshold，resize 的大小，等于  (capacity * load factor) ，达到 threshold 的时候就会扩容 (会 rehash，复制数据等操作，比较消耗性能)
- loadFactor，负载因子

## 功能实现 - method

### hash()

```java
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取 hashCode 值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

1. 取 key 的 hashCode 值
2. 高位运算
3. 取模运算

这个有一个点，就是 HashMap 中桶的个数 capacity 设计成 2 的幂次（**一般都是设计成素数的**）。之所以这样设计是为了在**取模**和**扩容**时做优化：

- 取模可以直接优化成了 and 位运算。
- 扩容时重新计算元素位置时取模更快了，而且扩容时可以直接根据 mask 位置上的新增位数是 0 还是 1 来确定当前元素是在【当前位置】，还是在【当前位置 * 2】的位置上。

但是这样设计也带来了一些问题，取模后的哈希值是有可能出现很差的哈希分布的，在 capacity 比较小的时候，hash 值高位 Bit 的信息完全丢失了。所以做了一个高位运算逻辑右移 `>>>` 16 位（逻辑右移，左边全部使用 0 填充），减少高位 Bit 信息的丢失。

### put()

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
- 判断 key 是否和当前结点的 key 相等，相等就直接替换 Node (或者 key 都为 null 的时候，HashMap 允许 key 为 null，顺便再插一句，HashMap 中 null 无法计算其 hash 值，默认都是放到下标为 0 的  上)
- 如果当前结点是 TreeNode 红黑树结点，就按红黑树的方法插入 (具体就不展开了)
- 如果是链表，就遍历链表，找到相同的 key 的 Node 就可以直接替换 Node，如果没找到就 newNode() (这个方法会在对应的 bin 中插入 Node)
- 统一替换 Node (即 e 不为空)
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
- 判断第一个 Node/TreeNode 的值是否为查询的 key，是的话直接返回 (always check first node) ，若不匹配就下一步
- 判断为 TreeNode，查找红黑树
- 判断链表，遍历链表查找

### resize()

扩容 (resize) 就是重新计算容量，向 HashMap 对象里不停的添加元素，而 HashMap 对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然 Java 里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组，就像我们用一个小桶装水，如果想装更多的水，就得换大水桶。**所以扩容就涉及到哈希值的重新计算，复制元素等操作，是很消耗性能的，如果能在开始就能确定好元素的个数，建议在一开始就设置好。**

当插完元素后，`size > threshold = capacity * Load factor`，扩容就开始了。

```java
 void resize(int newCapacity) {   //传入新的容量
     Entry[] oldTable = table;    //引用扩容前的 Entry 数组
     int oldCapacity = oldTable.length;
     if (oldCapacity == MAXIMUM_CAPACITY) {  //扩容前的数组大小如果已经达到最大 (2^30) 了
         threshold = Integer.MAX_VALUE; //修改阈值为 int 的最大值 (2^31-1)，这样以后就不会扩容了
         return;
     }

     Entry[] newTable = new Entry[newCapacity];  //初始化一个新的 Entry 数组
     transfer(newTable);                         //！！将数据转移到新的 Entry 数组里
     table = newTable;                           //HashMap 的 table 属性引用新的 Entry 数组
     threshold = (int)(newCapacity * loadFactor);//修改阈值
 }
```

transfer() 方法将原有 Entry 数组的元素拷贝到新的 Entry 数组里。

```java
 void transfer(Entry[] newTable) {
     Entry[] src = table;                   //src 引用了旧的 Entry 数组
     int newCapacity = newTable.length;
     for (int j = 0; j < src.length; j++) { //遍历旧的 Entry 数组
         Entry<K,V> e = src[j];             //取得旧 Entry 数组的每个元素
         if (e != null) {
             src[j] = null;//释放旧 Entry 数组的对象引用（for 循环后，旧的 Entry 数组不再引用任何对象）
             do {
                 Entry<K,V> next = e.next;
                 int i = indexFor(e.hash, newCapacity); //！！重新计算每个元素在数组中的位置
                 e.next = newTable[i]; //标记 [1]
                 newTable[i] = e;      //将元素放在数组上
                 e = next;             //访问下一个 Entry 链上的元素
             } while (e != null);
         }
     }
 }
```

可以看出来复制的时候，链表是用头插法的方式复制的，所以原来的链表会被 revert 一下。**在多线程场景，在两个线程 resize 的时候，可能会出现环形链表。**

## 扩展问题

### 1. Hash 时取模一定要模质数吗？

好的 hash 算法期望是 hash 值在取模后能尽可能均匀地分布。不模 prime number 的话，很容易会有分布不均匀的情况产生。

假设需要放入长为 m 的 hash 表的数是 n，取模函数是： $f(n) = n \% m$ ， $f(n)$ 为需要放置在 hash 表中的位置的下标。

取 m 为质数的目的是：让 m 和 n 的最大公约数为 1，也就是： $gcd(m, n)=1$ 。因为在 m 为质数时，除非 n 恰好是 m 的倍数，否则他们的最大公约数都为 1。

$$  f(n) = n \% m $$

$$  \Rightarrow a \times m + f(n) = n(a = 0, 1, 2,3...) $$

$$ \Rightarrow f(n) = n - a \times m $$

$$ \Rightarrow f(n) = gcd(m, n) *( \frac{n}{gcd(m,n)} - \frac{a \times m}{gcd(m,n)}) $$

也就是我们算出来的 hash 值只可能是为 $gcd(m, n)$ 的倍数的那些值，倘若数据特别一点，全都是 gcd 的倍数，这样会让分布很不均匀。

- [Hash 时取模一定要模质数吗？ - Porzy 的回答 - 知乎](https://www.zhihu.com/question/20806796/answer/231084056)

### 2. 为什么 java 在计算 hashcode 时使用 31 作为乘数？

确实很奇怪。。感觉是个 magic number。把这个搞成了一个类似 31 进制的数字。看了很多说法，stackoverflow 上说：

> The value 31 was chosen because it is an odd prime. If it were even and the multiplication overflowed, information would be lost, as multiplication by 2 is equivalent to shifting. The advantage of using a prime is less clear, but it is traditional. A nice property of 31 is that the multiplication can be replaced by a shift and a subtraction for better performance: 31 * i == (i << 5) - i. Modern VMs do this sort of optimization automatically.

是因为 31 是个奇素数，并且可以直接优化成位运算提高性能。偶数的话，相当于 shift 运算，会丢失信息，但是奇数因为在 shift 后在加上原数，所以比偶数好。之所以用素数是因为 traditional。。（我看到有的会用 33 来做，据说和 31 差不多）。

有个答案提到是根据实验得出发现 31 的，根据字典跑数据跑出的结果。（但是我看那个数据也是英文字典，ascii 字符，要是非 ascii 字符的实验结果呢？）

- [Why does Java's hashCode() in String use 31 as a multiplier? - Stack Overflow](https://stackoverflow.com/questions/299304/why-does-javas-hashcode-in-string-use-31-as-a-multiplier)
- [为什么Java String哈希乘数为31？](https://mp.weixin.qq.com/s/sCWQGU_OWiQkDUuSPXvw-w)


## 参考

- [Java 8 系列之重新认识 HashMap - 美团技术团队](https://tech.meituan.com/2016/06/24/java-hashmap.html)
- [HashMap? ConcurrentHashMap? 相信看完这篇没人能难住你！ | crossoverJie's Blog](https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/)