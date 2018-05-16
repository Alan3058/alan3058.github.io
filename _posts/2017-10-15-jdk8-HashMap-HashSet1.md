---
layout: post
title: [Java源码心得jdk8-HashMap和HashSet一]
categories: [Java]
tags: [jdk8,HashMap,HashSet,源码分析,源码心得]
id: [40532393412526080]
fullview: false
---

### HashMap源码分析

HashMap、HashSet和ArrayList一样，都是使用非常频繁的集合，理解这两个的实现原理，在使用的时候会更加得心应手。HashMap内部由两部分组成，首先是Node数组（Hash桶数组），每个Node元素（桶元素）又单独组成一个数据结构，可能是链表或者红黑树（当链表长度大于8时，将转换成红黑树。mark：红黑树原理将单独作为一个专题来学习），如下图。

![blob.png](/assets/resources/image/20171030/1509374380839078086.png "1509374380839078086.png")

在深入了解HashMap前，先说明以下几个概念。

* Hash桶（bucket）数组：HashMap的内部数组结构。

* 桶（bucket）元素：内部数组元素，每个元素又可组成一个链表或者红黑树。


* HashMap容量（table.length）：内部数组长度。默认情况下为16，之后每次扩容都会翻一倍。

* HashMap大小或者长度（size）：实际key/balue键值对数量。

* 加载因子（loadFactor）：主要是用来计算扩容极限值，加载因子\*容量=扩容极限值。

* 扩容极限值（threshold）：当HashMap长度大于扩容极限值，则集合开始扩容。


HashMap中有两个重要的数学运算，我把它们分别叫做hash和index运算。

第一次hash：key->hash，计算key的hash值。

```java
hash = (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
```

第二次index：hash->index，通过hash值计算在数组中的索引值。如下n为容量值

```java
index = (n - 1) & hash
```

### 初始化


HashMap的构造函数主要是初始化HashMap的容量值和加载因子。默认初始容量为16，加载因子为0.75。这里注意的是在构造函数中并不会立刻初始化内部数组，内部数组会在第一次put时，进行初始化，这样可以做到内存使用滞后。

```java
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; 
    }
```

### put方法

1.首先计算key的hash值。

2.通过key和容量值-1进行与运算操作，得到数组的索引值。

3.判断索引下的元素是否为树节点。如果是树节点的话，进行红黑树的新增/更新节点操作。

4.否则进行链表新增/更新节点操作。如果该链表长度大于8，则将链表转换成红黑树。

5.如果当前元素长度大于扩容极限值，则进行扩容操作。

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            //数组节点为空，首次扩容
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            //i索引下没有元素，该节点作为桶下第一个元素
            tab[i] = newNode(hash, key, value, null);
        else {
            //i索引下有元素，定位到桶的首节点
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                //新节点与首节点key值相等，将在后面更新value
                e = p;
            else if (p instanceof TreeNode)
                //key值不相等，如果首节点为树，则按红黑树处理
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //否则按链表方式处理，遍历链表
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        //找不到相同key节点，在末尾添加新节点
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            //此时节点长度大于8，也将链表转换成红黑树
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        //在链表中匹配到了key值相等的节点，跳出循环，再更新value值
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                //更新value值，并返回旧值
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

### resize方法

扩容方法，当集合长度大于扩容极限值时，将进行集合扩容操作，以减少元素hash碰撞概率。每次扩容容量都是原先的两倍，扩容极限值也为原先两倍。


由于容量发生了变化，所以扩容后还需要将原本的结构中的元素重新hash和index处理，得到新的数组索引值。reindex原理：将索引m下的元素的hash值与oldCap（旧容量）做与运算，结果为0则将元素置于索引m下，否则将元素置于索引m+oldCap下。

```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        //设置新的容量值和扩容极限值，新的容量值为旧的两倍，扩容极限值等于加载因子乘以新的容量值
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            //扩容，元素重组
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        //桶下只有一个节点是，直接hash和新容量值做与运算计算新的索引值
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        //树节点，进行红黑树拆分
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        //链表拆分
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                //相当于reIndex重新计算元素新的索引值，将原先链表拆分成两个链表
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

待续中。。。 [Java源码心得jdk8-HashMap和HashSet二](http://ctosb.com/article/40545367412346080)

