---
layout: post
title: [Java源码心得jdk8-HashMap和HashSet二]
categories: [Java]
tags: [jdk8,HashMap,HashSet,源码分析,源码心得]
id: [40545367412346080]
fullview: false
---
在第一篇[Java源码心得jdk8-HashMap和HashSet一](http://ctosb.com/article/40532393412526080)中已经了解了HashMap的大致原理和添加元素和扩容元素方法的源码。接下来继续看它的get、remove方法源码。

### get方法

1.通过key值计算hash值，再通过hash得到在数组中的索引值。

2.遍历树节点或者链表节点，得到目标元素。
final Node<K,V> getNode(int hash, Object key) { Node<K,V>[] tab; Node<K,V> first, e; int n; K k; //用hash和容量进行与运算得出索引值，得到桶 if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) { //桶头元素的key和入参key相等，则返回头元素 if (first.hash == hash && // always check first node ((k = first.key) == key || (key != null && key.equals(k)))) return first; if ((e = first.next) != null) { //遍历桶下子元素 if (first instanceof TreeNode) //如果是树结构，则遍历树 return ((TreeNode<K,V>)first).getTreeNode(hash, key); do { //遍历链表 if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) return e; } while ((e = e.next) != null); } } return null; }

### remove方法

1.通过key值计算hash值，再通过hash得到在数组中的索引值。

2.遍历树节点或者链表节点，得到目标元素。

3.进行树或者链表删除节点操作
final Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) { Node<K,V>[] tab; Node<K,V> p; int n, index; if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) { Node<K,V> node = null, e; K k; V v; //首先查找key对应的元素，类似get方法 if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) node = p; else if ((e = p.next) != null) { if (p instanceof TreeNode) node = ((TreeNode<K,V>)p).getTreeNode(hash, key); else { do { if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) { node = e; break; } p = e; } while ((e = e.next) != null); } } if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) { //找到key对应的节点，进行树或者链表删除操作 if (node instanceof TreeNode) ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable); else if (node == p) tab[index] = node.next; else p.next = node.next; ++modCount; --size; afterNodeRemoval(node); return node; } } return null; }

### HashSet原理

HashSet的实现很简单，它内置了一个HashMap属性。每当我们add(key)时，相当于map.put(key,new Object())操作，即存储一个key和Object对象。它的所有的操作都是对map进行操作。如下部分源代码
public HashSet() { map = new HashMap<>(); } public boolean add(E e) { return map.put(e, PRESENT)==null; } public boolean remove(Object o) { return map.remove(o)==PRESENT; }
