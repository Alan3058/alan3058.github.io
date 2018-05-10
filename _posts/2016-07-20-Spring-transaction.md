---
layout: post
title: [Spring事务]
categories: [Spring]
tags: [spring,spring事务]
id: [18826288693248]
fullview: false
---

# 背景

Spring框架是一个轻量级的IOC和AOP框架，使用Spring时，百分之九十九会使用Spring的事务功能。Spring的事务是Spring AOP典型的应用，通过JDK的动态代理或者CGLIB字节代理来实现拦截目标方法，并增强该方法功能。

在Spring的TransactionDefinition接口中为事务定义了4个属性：事务传播、事务隔离、超时时间、是否可读。

# Spring事务传播

Spring事务传播包含7个级别:require，require_new，nested，support，not_support，mandatory，never。默认情况下传播行为是require。

**require**:是默认情况下的传播行为，它要求必须要有事务。如果当前没有事务，则创建新事务;如果有事务，则加入到当前事务。

**require_new**:该要求必须要有事务。如果当前没有事务，则创建事务;如果有事务，也创建一个新事务，并且该事务无当前事务没有关系。

**nested**:该级别和require_new差不多，只不过新增的事务必须依赖当前的事务。

**support**:该级别是支持事务。如果当前事务存在时，则加入到当前事务，如果不存在，则没有事务。

**not_support**:该级别不支持事务。如果当前事务存在，则挂起当前事务。

**mandatory**:该级别表示必须要有事务，如果没有事务，则抛出异常。

**never**:该级别表示不支持事务，如果事务存在，则抛出异常。

# Spring事务隔离级别


为什么要事务隔离呢？没有事务隔离会发生什么问题呢？这里列出如下三个可能会造成的问题。

**脏读**：即A事务读取了B事务还未提交的数据，并且进行操作，然而之后B事务可能出现异常回滚，这样A事务读取的数据就无效。

**不可重复读**：即A事务第一次读取一份数据，这时B事务对这份数据进行了修改，当A事务第二次去读取这份数据的时候，两份数据不一致。

**幻读**：幻读和不可重复读基本上是一样的意思，它指A事务第一次读取了3条数据，这是B事务往数据库插入2条数据，当A事务第二次去读取时，读取到5条数据。

Spring事务隔离级别主要分以下5种，实际上只有4种：read_uncommited,read_commited,repeatable_read,serializable。

**default**:默认值，表示使用数据库默认的隔离级别，大部分数据库默认是read_commited级别。

**read_uncommited**：可以读取其他事务还未提交的数据，最低安全级级别。


**read_commited**：只能读取其他事务已经提交的数据，可以避免脏读问题。

**repeatable_read**：重复读取数据，数据一致，可以避免脏读和不可重复读的问题。

**serializable**：事务串行执行，安全级别最高，但是效率低，可以避免脏读、不可重复读和幻读。

脏读不可重复读幻读read_uncommited有有有read_commited无有有repeatable_read无无有serialable无无无


