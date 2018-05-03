---
layout: post
title: [Aspectj切入点语法]
categories: [Spring]
tags: [aspectj,语法,切入点]
id: [18624957906944]
fullview: false
---

AspectJ类型匹配通配符含义。

\*：匹配任何数量字符；

..：匹配任何数量字符的重复，如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数。

+：匹配指定类型的子类型；仅能作为后缀放在类型模式后边。

例子：

public \* \*(..) ：任何公共方法。

\* com..\*.\*(..)：com包以及所有子包下所有类的任何方法。

\* com..Manager.\*(..)：com包以及所有子包下的Manager类的任何方法。

\* com.alan..\*Manager.find\*(..)：com.alan包以及所有子包下的以Manager结尾的类的以find开头的任何方法。


