---
layout: post
title: [Java源码心得jdk8-CopyOnWriteArrayList和CopyOnWriteArraySet]
categories: [Java]
tags: [jdk8,CopyOnWriteArrayList,CopyOnWriteArraySet,源码分析,源码心得]
id: [40621131488559104]
fullview: false
---

### CopyOnWrite

CopyOnWrite（简写COW）是一种写时复制技术，Java中的CopyOnWriteArrayList和CopyOnWriteArraySet两个类就使用了CopyOnWrite技术。当往集合内部数组添加修改或者删除元素时，不会直接对原数组进行操作，而是先将容器复制一份出来，然后对新的容器进行操作，最后将原容器的引用指向新容器。

更多关于写时复制技术资料可查看[http://www.cnblogs.com/biyeymyhjob/archive/2012/07/20/2601655.html](http://www.cnblogs.com/biyeymyhjob/archive/2012/07/20/2601655.html) 

### CopyOnWriteArrayList源码分析

由于ArrayList是线程不安全，所以，故而在多线程环境下我们一般会选择CopyOnWriteArrayList去替代ArrayList，来保证程序数据正常。CopyOnWriteArrayList实现原理主要有以下几个点。

1.在内部数组属性上增加volatile关键字修饰，以保证内部数组的可见性和原子性，但不能保证数组元素的可见性和原子性。

2.增加了一把可重入锁，在添加、修改、删除操作进行加锁处理，在读操作中没有进行加锁处理。其中写操作（添加修改删除）都不是直接在原来的数组上进行操作，而且复制一份新数组，在新数组上进行。

基于每次写操作都需要复制一份新数组，额外的增加了一部分内存消耗，并且需要进行加锁同步处理，造成每次写的代价比较大，相反读操作的代价比较小。因此它更加适用于读多写少的应用场景下，并且推荐批量写操作来代替多次写，以减少多次写操作带来的消耗。

### 构造函数

构造函数主要是对内部数组进行初始化，默认数组长度为0。

```java
/**
 * Sets the array.
 */
final void setArray(Object[] a) {
    array = a;
}

/**
 * Creates an empty list.
 */
public CopyOnWriteArrayList() {
    setArray(new Object[0]);
}
```

### add方法

首先将旧的数组复制一份到新数组中，然后将新元素加到新数组中，最后将新数组赋值给内部数组属性，并且整个处理过程需要加锁。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        //将旧数组复制一份到新数组中
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        //将新元素添加到新数组尾部
        newElements[len] = e;
        //内部数组属性指向新数组
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

### get方法

直接获取内部数组对应索引的元素，该过程没有进行加锁处理，因此它返回的元素可能不是最新的数据。


```java
private E get(Object[] a, int index) {
    return (E) a[index];
}

/**
 * {@inheritDoc}
 *
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    return get(getArray(), index);
}
```

### remove方法

该方法实现方式和add方法类似，将索引前后的元素直接拷贝到新数组中，最后将新数组赋值给内部数组属性，整个过程需要加锁处理。


```java
public E remove(int index) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        if (numMoved == 0)
            //移出的元素是最后一个，则只复制索引前的元素到新数组中
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            Object[] newElements = new Object[len - 1];
            //复制索引前的元素到新数组中
            System.arraycopy(elements, 0, newElements, 0, index);
            //复制索引后的元素到新数组中
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            //内部数组属性指向新数组
            setArray(newElements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}
```

### iterator方法

该方法返回一个COWIterator迭代器实例，它遍历的是旧的内部数组，或者说遍历内部数组的一个快照，所以它迭代出来的数据可能不是最新的。如下是COWIterator迭代器的构造函数


```java
private COWIterator(Object[] elements, int initialCursor) {
    cursor = initialCursor;
    snapshot = elements;
}
```

### CopyOnWriteArraySet原理

CopyOnWriteArraySet的实现借助了CopyOnWriteArrayList类，它的内部包含一个CopyOnWriteArrayList对象al，它的实现方法都是调用al对象的方法。如下是部分源代码

```java
public CopyOnWriteArraySet() {
    al = new CopyOnWriteArrayList<E>();
}
public int size() {
    return al.size();
}
public boolean add(E e) {
        //去重，只有元素不存在，才添加
    return al.addIfAbsent(e);
}
public boolean remove(Object o) {
    return al.remove(o);
}
```


