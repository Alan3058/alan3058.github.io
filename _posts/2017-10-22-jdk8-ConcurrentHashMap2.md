---
layout: post
title: [Java源码心得jdk8-ConcurrentHashMap二]
categories: [Java]
tags: [jdk8,ConcurrentHashMap,源码分析,源码心得]
id: [46545554779406336]
fullview: false
---

接着第一篇[Java源码心得jdk8-ConcurrentHashMap一](/171021/jdk8-ConcurrentHashMap1)继续看transfer扩容方法源码，如下

### transfer方法

该方法为集合扩容方法，相当于HashMap的resize方法。相比之下它的处理原理和HashMap基本差不多，但是由于需要进行多线程处理，故而代码实现上更加复杂。

ConcurrentHashMap每次扩容操作和HashMap一样，扩容大小都为原先的两倍。ConcurrentHashMap扩容没有对整个桶元素进行加锁处理，而是只对当前桶元素进行加锁处理，并且每一次扩容，只对当前桶元素的子节点进行扩容，对其他桶元素的写操作毫无影响。每次扩容完会将当前桶元素替换为ForwardingNode对象节点，该节点的特征为hash值为-1，key和value值为null，但它有一个nextTable属性，指向新扩容数组结构。

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    if (nextTab == null) {            // initiating
        //nextTab为空，表示第一个线程进来扩容
        try {
            //每次扩容大小都为原来两倍
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        //扩容从内部数组最后一个桶元素的子节点开始。
        transferIndex = n;
    }
    int nextn = nextTab.length;
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
              //计算扩容元素的索引值
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) {
                //集合已完成扩容操作，结束扩容
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
              //其他线程发现扩容已完成，结束扩容
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        else if ((f = tabAt(tab, i)) == null)
            //桶元素为空时，扩容
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            //桶元素正在扩容，跳过，对下一个桶元素进行扩容
            advance = true; // already processed
        else {
            //实际扩容操作,对当前桶元素进行加锁。这样集合元素扩容分n个锁，可以多个线程并发扩容，当前桶元素扩容不会影响其他桶元素的操作，极大减少锁冲突概率。
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                        //链表扩容操作
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
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
                            //和HashMap处理方式一样，将索引i的桶元素上的节点，拆分到i和i+n两个桶元素上
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        //树形结构扩容操作
                        ......
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
                        ......
                    }
                }
            }
        }
    }
}
```

待续中。。。 [Java源码心得jdk8-ConcurrentHashMap三](/171022/jdk8-ConcurrentHashMap3)

