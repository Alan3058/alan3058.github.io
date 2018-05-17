---
layout: post
title: [Java源码心得jdk8-ConcurrentHashMap三]
categories: [Java]
tags: [jdk8,ConcurrentHashMap,源码分析,源码心得]
id: [46546116891639808]
fullview: false
---

在前两篇中已讲了ConcurrentHashMap添加元素和扩容源码的大致实现。接下来继续看get和remove方法。

[Java源码心得jdk8-ConcurrentHashMap一](/171021/jdk8-ConcurrentHashMap1)

[Java源码心得jdk8-ConcurrentHashMap二](/171022/jdk8-ConcurrentHashMap2)

### addCount方法

该方法主要在为put方法内末尾调用，主要完成两件事情。

1.更新统计元素数量

2.判断是否需要进行扩容
```java
private final void addCount(long x, int check) {
   CounterCell[] as; long b, s;
   //更新元素数量baseCount
   if ((as = counterCells) != null ||
         !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
      CounterCell a; long v; int m;
      boolean uncontended = true;
      if (as == null || (m = as.length - 1) &lt; 0 ||
            (a = as[ThreadLocalRandom.getProbe() &amp; m]) == null ||
            !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
         //新增一个CounterCell对象去计数
         fullAddCount(x, uncontended);
         return;
      }
      if (check &lt;= 1)
         return;
      s = sumCount();
   }
   //检查是否需要进行扩容
   if (check &gt;= 0) {
      Node&lt;K,V&gt;[] tab, nt; int n, sc;
      while (s &gt;= (long)(sc = sizeCtl) &amp;&amp; (tab = table) != null &amp;&amp;
            (n = tab.length) &lt; MAXIMUM_CAPACITY) {
         int rs = resizeStamp(n);
         if (sc &lt; 0) {
            if ((sc &gt;&gt;&gt; RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                  sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                  transferIndex &lt;= 0)
               break;
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
               //sizeCtl小于0，表示集合已经在扩容，则直接对nextTable进行扩容
               transfer(tab, nt);
         }
         else if (U.compareAndSwapInt(this, SIZECTL, sc,
               (rs &lt;&lt; RESIZE_STAMP_SHIFT) + 2))
            //没有其他现在在扩容，则新增nextTable进行扩容
            transfer(tab, null);
         s = sumCount();
      }
   }
}
```

### 

### get方法

1.通过key计算hash值。

2.通过hash计算index索引值。

3.找到桶数组对应index索引的桶元素。

4.如果桶元素是要找的node节点值，返回node节点value。

5.否则遍历桶元素子节点node(链表或者树结构)，寻找key值相等的node节点。

ConcurrentHashMap的get方法和HashMap的get方法实现原理基本相似，ConcurrentHashMap的不同是通过Unsafe的原子操作方法去获取桶元素。
```java
public V get(Object key) {
    Node&lt;K,V&gt;[] tab; Node&lt;K,V&gt; e, p; int n, eh; K ek;
    //计算hash值
    int h = spread(key.hashCode());
    if ((tab = table) != null &amp;&amp; (n = tab.length) &gt; 0 &amp;&amp;
        (e = tabAt(tab, (n - 1) &amp; h)) != null) {
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null &amp;&amp; key.equals(ek)))
                //桶下找到该key值对应的节点，直接返回值
                return e.val;
        }
        else if (eh &lt; 0)
                //hash值为负数，遍历树查找节点
            return (p = e.find(h, key)) != null ? p.val : null;
        //遍历链表查找节点
        while ((e = e.next) != null) {
            if (e.hash == h &amp;&amp;
                ((ek = e.key) == key || (ek != null &amp;&amp; key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

### remove方法

1.通过key计算hash值。

2.通过hash计算index索引值。

3.找到桶数组对应index索引的桶元素。如果此时集合正在扩容，则先帮助集合扩容，再去删除节点。

4.如果桶元素是链表节点，则遍历链表删除节点。

5.否则遍历树结构，删除节点。

该方法和HashMap的删除节点的方法实现原理基本差不多，但是ConcurrentHashMap的删除方法多了个帮助集合扩容步骤，并且使用了Unsafe原子方法获取和设置桶元素。
```java
public V remove(Object key) {
    return replaceNode(key, null, null);
}
final V replaceNode(Object key, V value, Object cv) {
    int hash = spread(key.hashCode());
    for (Node&lt;K,V&gt;[] tab = table;;) {
        Node&lt;K,V&gt; f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0 ||
            (f = tabAt(tab, i = (n - 1) &amp; hash)) == null)
            break;
        else if ((fh = f.hash) == MOVED)
            //删除过程遇到集合正在扩容，则先帮助扩容后，再删除结点
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            boolean validated = false;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh &gt;= 0) {
                        validated = true;
                        for (Node&lt;K,V&gt; e = f, pred = null;;) {
                            K ek;
                            if (e.hash == hash &amp;&amp;
                                ((ek = e.key) == key ||
                                 (ek != null &amp;&amp; key.equals(ek)))) {
                                V ev = e.val;
                                if (cv == null || cv == ev ||
                                    (ev != null &amp;&amp; cv.equals(ev))) {
                                    oldVal = ev;
                                    if (value != null)
                                        //替换对应节点的值
                                        e.val = value;
                                    else if (pred != null)
                                        //桶元素子节点是要删除的目标，则前一个节点指向目标指向的后一个节点。
                                        pred.next = e.next;
                                    else
                                        //桶元素是要删除的节点，设置后面的元素为桶元素
                                        setTabAt(tab, i, e.next);
                                }
                                break;
                            }
                            pred = e;
                            if ((e = e.next) == null)
                                break;
                        }
                    }
                    else if (f instanceof TreeBin) {
                        //遍历树，删除节点
                        validated = true;
                        TreeBin&lt;K,V&gt; t = (TreeBin&lt;K,V&gt;)f;
                        TreeNode&lt;K,V&gt; r, p;
                        if ((r = t.root) != null &amp;&amp;
                            (p = r.findTreeNode(hash, key, null)) != null) {
                            V pv = p.val;
                            if (cv == null || cv == pv ||
                                (pv != null &amp;&amp; cv.equals(pv))) {
                                oldVal = pv;
                                if (value != null)
                                    p.val = value;
                                else if (t.removeTreeNode(p))
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }
            if (validated) {
                if (oldVal != null) {
                    if (value == null)
                        addCount(-1L, -1);
                    return oldVal;
                }
                break;
            }
        }
    }
    return null;
}
```

