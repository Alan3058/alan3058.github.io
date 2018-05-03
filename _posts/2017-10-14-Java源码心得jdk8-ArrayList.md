---
layout: post
title: [Java源码心得jdk8-ArrayList]
categories: [Java]
tags: [jdk8,ArrayList,源码分析,源码心得]
id: [38851080209039360]
fullview: false
---
### ArrayList源码分析

ArrayList的源码实现应该算是Java集合工具类是最简单的一个，也是我们使用最多的一个集合类。它的内部实现是一个数组，在首次加入元素时，我们会创建一个默认大小（或者用户指定发现）的数组，默认值为16，之后的添加、删除、查询操作都是对该数组进行操作。

### 初始化

ArrayList提供了三个构造函数作为初始化入口，一个无参，一个接收用户定义初始化数组长度，另一个是接收一个集合Collection，将入参集合元素转换成数组元素。默认的构造函数会初始化设置ArrayList长度为16。以下是无参构造函数源码
public ArrayList() { this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA; }

### add方法

ArrayList提供四个添加元素的方法，这几个方法实现逻辑基本一致。首先需要确保数组容量是否充足，不足的话扩容当前容量的1/2倍，以后就是添加数组对应索引元素值。部分源码如下
public void add(int index, E element) { //检查索引是否越界 rangeCheckForAdd(index); //确保数组容量足够 ensureCapacityInternal(size + 1); // Increments modCount!! //索引之后的元素向后自动移动一位 System.arraycopy(elementData, index, elementData, index + 1, size - index); //赋值指定索引元素 elementData[index] = element; size++; } private void grow(int minCapacity) { // overflow-conscious code int oldCapacity = elementData.length; //新容量值最多为原先的1.5倍 int newCapacity = oldCapacity + (oldCapacity >> 1); if (newCapacity - minCapacity < 0) newCapacity = minCapacity; if (newCapacity - MAX_ARRAY_SIZE > 0) //新容量值不能超过设定的最大值，以防止下次扩容超越Integer的最大范围 newCapacity = hugeCapacity(minCapacity); // minCapacity is usually close to size, so this is a win: elementData = Arrays.copyOf(elementData, newCapacity); } private static int hugeCapacity(int minCapacity) { if (minCapacity < 0) // overflow throw new OutOfMemoryError(); return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE; }

### get方法

主要思想：直接从内部数组通过索引定位元素，然后返回。部分源码如下
public E get(int index) { //检查索引是否越界 rangeCheck(index); //获取元素 return elementData(index); } E elementData(int index) { return (E) elementData[index]; }

### remove方法

主要思想：先将要删除的索引后续元素往前移动一位，再将最后一位数组元素设为空（提醒gc回收）。部分源代码如下
public E remove(int index) { //检查索引值 rangeCheck(index); modCount++; E oldValue = elementData(index); int numMoved = size - index - 1; if (numMoved > 0) //删除的元素后还有元素，则将后面的元素往前移一位 System.arraycopy(elementData, index+1, elementData, index, numMoved); elementData[--size] = null; // clear to let GC do its work return oldValue; }

### iterator方法

该方法会返回一个迭代器Iterator，之后可以通过迭代器的next方法更加便利去访问集合。迭代器实现思想其实比较简单，它在内部通过一个游标变量保存当前访问数组元素的索引，每次访问时游标自增1。部分源代码如下
private class Itr implements Iterator<E> { int cursor; // index of next element to return int lastRet = -1; // index of last element returned; -1 if no such int expectedModCount = modCount; public boolean hasNext() { return cursor != size; } @SuppressWarnings("unchecked") public E next() { checkForComodification(); int i = cursor; if (i >= size) throw new NoSuchElementException(); Object[] elementData = ArrayList.this.elementData; if (i >= elementData.length) throw new ConcurrentModificationException(); cursor = i + 1; //返回下一个数组元素 return (E) elementData[lastRet = i]; } .... }

### listIterator方法

该方法会返回一个迭代器ListIterator，它是Iterator的子类，它增加了向前遍历的方法和向头向尾添加元素的方法。它的实现思想和Iterator一样，也是在内部有一个游标变量去保存当前遍历元素在数组的索引值。部分源代码如下
private class ListItr extends Itr implements ListIterator<E> { ListItr(int index) { super(); cursor = index; } public boolean hasPrevious() { return cursor != 0; } public int nextIndex() { return cursor; } public int previousIndex() { return cursor - 1; } @SuppressWarnings("unchecked") public E previous() { checkForComodification(); int i = cursor - 1; if (i < 0) throw new NoSuchElementException(); Object[] elementData = ArrayList.this.elementData; if (i >= elementData.length) throw new ConcurrentModificationException(); cursor = i; //返回前一个数组元素 return (E) elementData[lastRet = i]; } ... }
