---
layout: post
title: [Java源码心得jdk8-ConcurrentHashMap一]
categories: [Java]
tags: [jdk8,ConcurrentHashMap,源码分析,源码心得]
id: [40694842942029824]
fullview: false
---

### ConcurrentHashMap源码分析

ConcurrentHashMap是HashMap在高并发场景下的实现。它的内部结构和HashMap基本一致，也是由一个内部数组，外加一个链表或者树形结构组成。

![blob.png](http://file.ctosb.com/upload/image/20171030/1509374380839078086.png "1509374380839078086.png")

它通过CAS机制来保证并发写操作时数据正确。在它的内部添加了一个Unsafe对象U，所有的原子读操作和写操作都是通过U对象操作。Unsafe类的实现原理是，通过获取对象的地址+字段相对对象地址的偏移量地址，已完成准确更新和获取对象。关于Unsafe类实现可以搜索它的Java和C实现文件，以便参考。以下是ConcurrentHashMap使用Unsafe的三个方法：读、CAS写、写。


```java
//获取对应索引的node节点
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
//CAS设置node数组对应索引的值
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
//设置node数组对应索引的对象值
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}
```

在Java新版本api的并发类中，有许多类都是通过Unsafe类去实现，比如AtomicInteger、AtomicBoolean、AtomicLong、CountDownLatch等类，都用到了Unsafe类方法。

### 重要属性

1.table：存储内部数组Node，集合实际存储数据的结构。

2.sizeCtl：该属性和HashMap的扩容极限值threshold类似，它能控制容器扩容的时机。不仅如此，它不同的值，包含不同的含义。

为-1时：标识该容器正在初始化。

* 小于-1：即-n，表示容器正在变更容量，n为进行变更容量的线程数。

* 0：默认值。


* 正数：表示下一次要扩容的极限值（等于0.75/*容量值），即容量大于该值，则进行扩容。另一层含义，在初始化构造函数时，如果有初始化长度参数，它会暂时代表初始化长度。


3.nextTable：node数组，只有在扩容时，该值不为空，它表示扩容后新的内部数组，扩容完成会将table变量引用指向它。

4.transferIndex：表示下一次扩容的内部数组元素的索引值。

5.baseCount：在未发生多线程前提下集合内部元素的数量。


6.counteCells：CounterCell对象数组。多线程并发写时，会将每个线程会将各自写的元素数量写入到CounterCell对象中。最终元素数量为baseCount+sum(counterCells)。

### put方法


put方法和HashMap的实现原理大致相同，只是在添加、修改节点时，通过Unsafe类的方法去更新的。在put方法中有两个重要的方法步骤，第一个是在添加修改节点的情况下，如果发现当前集合正在被其他线程进行变更容量操作时，那么当前线程将会调用helpTransfer方法去辅助变更容量操作。增加完元素后将会试图去更新当前元素总数量，执行addCount方法。

注意：在添加元素时，不会对整个内部数组进行加锁处理或者CAS更新，而是对桶数组的某一个数组元素进行加锁，只会对该数组元素的子节点结构（链表、树）的写操作进行同步。即将内部数字元素分段处理写操作，减少锁冲突概率。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    //计算hash值，和HashMap差不多
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            //首次添加，初始化内部node数组
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            //桶下索引i元素不存在，新增节点(采用cas机制)
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            //当有其他线程在进行扩容、缩容操作，则帮助集合进行扩容、缩容操作
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            //加锁遍历桶的子节点，该桶可能是链表或者红黑树。
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        //hash值为正数，则为链表，开始遍历链表
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                //链表中找到该key值的节点，判断是否需要更新value值
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                //链表中不存在该key对应的节点，则在尾部插入新节点
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        //hash值为负数，则为树的根节点()
                        Node<K,V> p;
                        binCount = 2;
                        //执行树结构添加节点操作
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
                    //链表长度大于8，则将链表转换成树
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

### helpTransfer方法

当桶元素节点是ForwardingNode，则调用transfer方法进行去辅助其他线程的扩容操作。此时sizeCtl为负数。


```java
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
   Node<K,V>[] nextTab; int sc;
   //当桶元素节点为ForwardingNode时，则进行辅助扩容操作
   if (tab != null && (f instanceof ForwardingNode) &&
         (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
      int rs = resizeStamp(tab.length);
      while (nextTab == nextTable && table == tab &&
            (sc = sizeCtl) < 0) {
         if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
               sc == rs + MAX_RESIZERS || transferIndex <= 0)
            break;
         if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
            //辅助其他线程进行扩容操作
            transfer(tab, nextTab);
            break;
         }
      }
      return nextTab;
   }
   return table;
}
```

待续中。。。 [Java源码心得jdk8-ConcurrentHashMap二](http://ctosb.com/article/46545554779406336)

