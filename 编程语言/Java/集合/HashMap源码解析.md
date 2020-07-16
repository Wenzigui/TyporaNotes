# HashMap源码解析

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083509918-5ae57596-ae6b-4779-9a1e-a4518567b3a9.png#align=left&display=inline&height=526&originHeight=526&originWidth=1194&size=0&status=done&style=none&width=1194)

# 1. 概述

HashMap是基于hash表的Map接口实现，允许null key、null value

# 2. 成员变量

```java
/**
     * [1] 默认初始容量，16（必须是2的幂）
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    
    /**
     * 最大容量
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;
    
    /**
     * 默认的负载因子
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    
    /**
     * 链表转红黑树的阈值
     */
    static final int TREEIFY_THRESHOLD = 8;
    
    /**
     * [2] 红黑树转列表的阈值
     */
    static final int UNTREEIFY_THRESHOLD = 6;
    
    /**
     * 最小树形化阈值
     * 当HashMap中的table的长度大于64的时候，这时候才会允许桶内的链表转成红黑树（要求桶内的链表长度达到8）
     * 如果只是桶内的链表过长，而table的长度小于64的时候
     * 此时应该是执行resize方法，将table进行扩容，而不是链表转红黑树
     * 最小树形化阈值至少应为链表转红黑树阈值的四倍
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    /**
     * 存放具体元素的集
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;
    
    /**
     * HashMap的负载因子
     * 负载因子控制这HashMap中table数组的存放数据的疏密程度
     * 负载因子越接近1，那么存放的数据越密集，导致查找元素效率低下
     * 负载因子约接近0，那么存放的数据越稀疏，导致数组空间利用率低下
     * The load factor for the hash table.
     *
     * @serial
     */
    final float loadFactor;
    
    /**
     * 修改次数
     */
    transient int modCount;
    
    /**
     * 键值对的个数
     * The number of key-value mappings contained in this map.
     */
    transient int size;
    
    /**
     * 存储元素的数组
     */
    transient Node<K,V>[] table;
    
    /**
     * 当{@link HashMap#size} >= {@link HashMap#threshold}的时候，数组要进行扩容操作
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    int threshold;
```

[1] 处，为什么会要求`HashMap`的容量必须是2的幂，可以看看

[HashMap 容量为2次幂的原因](https://blog.csdn.net/eaphyy/article/details/84386313)

# 3. 构造方法

## java.util.HashMap#HashMap()

```java
public HashMap() {
        // 使用默认的参数
        // 默认的负载因子、默认的容量
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```

默认的构造函数里面并没有对`table`数组进行初始化，这个操作是在`java.util.HashMap#putVal`方法进行的

## java.util.HashMap#HashMap(int)

```java
public HashMap(int initialCapacity) {
        // 调用重载构造函数
        // 指定初始容量，默认的负载因子
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```

## java.util.HashMap#HashMap(int, float)

```java
public HashMap(int initialCapacity, float loadFactor) {
        // 指定初始容量，指定负载因子
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);

        if (initialCapacity > MAXIMUM_CAPACITY)
            // 指定的初始容量大于最大容量，则取最大容量
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            // 检查负载因子
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        // 手动指定HashMap的容量的时候，HashMap的阈值设置跟负载因子无关
        this.threshold = tableSizeFor(initialCapacity); // [1]
    }
```

`tableSizeFor`这个方法的作用是，根据指定的容量，大于指定容量的最小的2幂的值

比如说，给定15，返回16；给定30，返回32

> `tableSizeFor`方法是一个很牛逼的方法，5行代码看得我一脸懵逼


## java.util.HashMap#HashMap(java.util.Map<? extends K,? extends V>)

```java
public HashMap(Map<? extends K, ? extends V> m) {
        // 使用默认的负载因子
        this.loadFactor = DEFAULT_LOAD_FACTOR;

        putMapEntries(m, false);
    }
    
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            // 判断table是否已经实例化
            if (table == null) { // pre-size
                // 计算m的扩容上限
                float ft = ((float)s / loadFactor) + 1.0F;
                // 检查扩容上限是否大于HashMap的最大容量
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold) {
                    // m的扩容上限大于当前HashMap的扩容上限，则需要重新调整
                    threshold = tableSizeFor(t);
                }
            }
            else if (s > threshold)
                // m.size大于扩容上限，执行resize方法，扩容table
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                // 将m中所有的键值对添加到HashMap中
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

`putMapEntries`方法的流程：

1. 如果table为空，则重新计算扩容上限
1. 如果HashMap的扩容上限小于指定Map的`size`，那么执行`resize`进行扩容
1. 将指定Map中所有的键值通过`putVal`方法放到HashMap中

> 这里使用到的`hash`函数实际上是一个**扰动函数**，下文会介绍的


# 4. Very重要的方法

## java.util.HashMap#tableSizeFor

```java
/**
     * 静态工具方法
     * 根据指定的容量，大于指定容量的最小的2幂的值
     * 备注：牛逼的算法
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        /*
        算法的原理请看
        HashMap源码注解 之 静态工具方法hash()、tableSizeFor()（四） - 程序员 - CSDN博客
        https://blog.csdn.net/fan2012huan/article/details/51097331
         */
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

算法的流程如下

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083509878-cda22a6b-8adc-4f0a-8e36-10f2ca0ab120.png#align=left&display=inline&height=1330&originHeight=1330&originWidth=2115&size=0&status=done&style=none&width=2115)

> 位移的那五句代码是真的很牛逼！！如果看完流程图以后，还不懂，那就看看注释里面的文章吧


## java.util.HashMap#hash

```java
// 扰动函数
    static final int hash(Object key) {
        int h;
        // 混合原始Hash值的高位和地位，减少低位碰撞的几率
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

为什么会有这个函数出现？

首先我们要了解，HashMap是根据key的hash值中几个低位的值来确定key在table中对应的index

这句话怎么理解呢？我举个栗子

有一个32位的hash值如下

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083509921-518e21ae-94c3-4f41-b65f-2167d28e577d.png#align=left&display=inline&height=1553&originHeight=1553&originWidth=2402&size=0&status=done&style=none&width=2402)

如果取Hash值的低4位，则index = 0101 = 5

如果出现大量的低4位为0101的hash值，那么所有键值对都会放在table的index = 5的地方

这样就会导致**key无法均匀分布在table中**

那么HashMap为了解决这个问题，就搞出了这个方法`java.util.HashMap#hash`

把一个32位的hash值的高16位 & 低16位，那么**低位就会携带高位的信息**

说白了就是，即使有大量hash值低位相同的key，经历过hash方法后，计算得到的index会不一样

> 通过hash方法降低hash冲突的概率


## java.util.HashMap#resize

```java
/**
     * 对table初始化操作或扩容操作
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        // 拿出旧table快照
        Node<K,V>[] oldTab = table;
        // 检查旧table容量
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        // 旧的扩容上限
        int oldThr = threshold;
        // 新table的容量和扩容上限
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                // 如果旧table的容量大于HashMap的最大容量，则不进行扩容操作了
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                // 没有超过HashMap的最大容量，则扩容两倍（newCap，newThr）
                newThr = oldThr << 1; // double threshold
        }

        else if (oldThr > 0) // initial capacity was placed in threshold
            // 只设置了扩容上限，没有初始化table，将初始容量设置为扩容上限
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 没有设置扩容上限，没有初始化table，则使用默认的容量（16）和扩容上限（12）
            // 比如：java.util.HashMap.HashMap()
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            // newThr在oldCap > 0的条件扩容两倍后仍然等于0，那就说明，oldThr原本就是0
            // 重新计算新的扩容上限
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 更新HashMap的扩容上限
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        // 初始化table
        table = newTab;
        // 如果old table不为空，则需要将里面的
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        // 链表上只有一个节点，根据节点hash值，重新计算e在newTab中的index
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        // 链表已经转成红黑树，拆分树
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        // 定义两个链表，lo链表和hi链表
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // 通过hash值&oldCap判断链表上的节点是否应该停留新table中的原位置
                            if ((e.hash & oldCap) == 0) {
                                // 节点仍然停留在位置j
                                // 插入lo链表（尾插法）
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                // 节点转移到位置j+oldCap
                                // 插入hi链表（尾插法）
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            // 如果lo链表非空, 把整个lo链表放到新table的j位置上
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            // 如果hi链表非空，把整个hi链表放到新table的j+oldCap位置上
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

`resize`流程并不复杂，大致如下

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083509892-d5edd2c3-d51d-4da9-9a2b-b528615f7968.png#align=left&display=inline&height=1285&originHeight=1285&originWidth=1059&size=0&status=done&style=none&width=1059)

个人认为，比较关键的一点是**重新分配键值对到新table**

这个时候要考虑三种情况：

1. table中index位置只有一个元素
1. table中index位置上是一棵红黑树
1. table中index位置上是一条链表（**重点看这个**）

如果是第3种情况，table中index位置上是一条链表，再重新分配的时候，会把这个链表拆分成两条链表

一条lo链表，留在原来的index位置

另一条hi链表，会被移动到`index+oldCapacity`的位置

此时，需要判断**节点是留在lo链表，还是放在hi链表？**

推荐看一下这篇文章 [深入理解HashMap(四): 关键源码逐行分析之resize扩容](https://segmentfault.com/a/1190000015812438?utm_source=tag-newest)

> 在jdk1.7里，table的扩容在多线程并发执行下会形成环，墙裂推荐仔细阅读这篇文章🎉🎉 [HashMap链表成环的原因和解决方案](https://www.douban.com/note/734486437/)


## java.util.HashMap#treeifyBin

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            // [1] table的长度小于最小树形化阈值，执行resize方法扩容
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            // 将链表转成红黑树
            // hd头节点、tl尾节点
            TreeNode<K,V> hd = null, tl = null;
            // do-while循环将单向链表转成双向链表
            do {
                // 将节点转成TreeNode
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    // 设置根节点
                    hd = p;
                else {
                    // 将树节点的前一个节点指向尾节点
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            // 将双向链表转成红黑树
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

`treeifyBin`方法的作用是将table中某个index位置上的链表转成红黑树

这个方法一般是在添加或合并元素后，发现链表的长度大于`TREEIFY_THRESHOLD`的时候调用

[1]处可以看到，如果当前`table.length`小于最小树形化阈值（64），那么会调用resize方法进行扩容，而不是将链表树形化

## java.util.HashMap#putVal

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            // [1] table还没有初始化，通过resize方法初始化table
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            // 如果key的hash值在table上对应的位置没有元素，则直接将创建节点
            tab[i] = newNode(hash, key, value, null);
        else {
            // table上对应的位置已经存在节点
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                // 指定的key与已存在的节点的key相等
                e = p;
            else if (p instanceof TreeNode)
                // 链表节点已变成红黑树节点
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 遍历链表，插入或更新节点
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        // 在链表的尾部新添加一个节点
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            // TREEIFY_THRESHOLD - 1的原因是binCount是从0开始，链表上有8个节点的时候，binCount=7
                            // 添加节点后，当链表的节点数量大于等于8的时候，将链表树形化
                            // [2]
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        // 在链表中发现了key对应的节点
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                // 发现了key对应的节点，则更新节点上的value
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    // 更新节点对应的值
                    e.value = value;
                afterNodeAccess(e);
                // 返回节点的旧值
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            // [3]
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

`putVal`方法是添加键值对相关方法的实现

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083509901-8ee59253-5600-4012-88ee-75f0a05325eb.png#align=left&display=inline&height=167&originHeight=167&originWidth=989&size=0&status=done&style=none&width=989)

上图可以看到，添加键值对的方法内部都会调用`putVal`方法

[1]处可以看到，`putVal`方法内部通过调用`resize`方法对table进行初始化

整体逻辑并不复杂，但是要注意一下[2]、[3]处

[2]处，如果添加节点后，链表过长，要将链表转成红黑树

[3]处，如果添加节点后，整个HashMap的键值对数量达到了扩容上限，那么要对table进行扩容操作

# 5. 总结

如果说ArrayList的源码阅读难度是一星半，那么我觉得HashMap的源码阅读难度至少有三颗星

这篇文章省略了一些内容，比如HashMap里面的红黑树实现，不写上去的原因主要是我也不是很懂红黑树😂，后续的时间如果我弄懂了，我会再补一篇

最起码这篇文章把HashMap最重要的几个方法的实现讲得比较明白，还是可以的😁

# 6. 推荐阅读

[HashMap源码注解 之 静态工具方法hash()、tableSizeFor()](https://blog.csdn.net/fan2012huan/article/details/51097331)

[深入理解HashMap(四): 关键源码逐行分析之resize扩容](https://segmentfault.com/a/1190000015812438?utm_source=tag-newest)

[HashMap中的hash函数 - 淡腾的枫 - 博客园](https://www.cnblogs.com/zhengwang/p/8136164.html)

[HashMap 容量为2次幂的原因](https://blog.csdn.net/eaphyy/article/details/84386313)

[HashMap链表成环的原因和解决方案](https://www.douban.com/note/734486437/)
