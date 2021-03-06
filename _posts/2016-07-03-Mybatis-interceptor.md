---
layout: post
title: [Mybatis拦截器]
categories: [Mybatis]
tags: [mybatis,拦截器]
id: [18796928565248]
fullview: false
---

# 背景

最近在整合Mybatis、Spring，欲通过他们来改造下以前自己做过的项目，来巩固下自己的系统知识。在这里并没有选用Hibernate、JPA等ORM框架，为什么我不选用他们呢？以下仅是个人对Hibernate和Mybatis见解，如果不正，欢迎指正。

HibernateMybatis全自动化ORM框架半自动化ORM框架纯面向对象编程面向Sql表与Model的关系映射结果集与Model的关系映射入门门槛低，深入学习不易入门门槛稍高，深入学习简单基本上只需要会配置注解即可，可不用写sql，也可写sql，大部分情况下不需要必须写sql执行效率不高效率高，仅次于spring Jdbc一套代码，适用所有数据库需针对每套数据库修改遇到问题不好处理，需深入了解底层简单，只是对jdbc的简单封装，容易排查问题

# Mybatis拦截器

拦截器是个很通用的概念，基本上在很多框架上都有用上，比如Spring、Mybatis等框架，特别是在Spring中已经是炉火纯青了。


Mybatis提供了四个拦截的接口，可拦截Executor、ParameterHandler、StatementHandler、ResultSetHandler这四个接口的方法，Mybatis本身没有提供默认的拦截器实现，需要开发者自己实现。

以下是这四个接口的拦截顺序，纯属copy，还未验证，但感觉应该差不多吧。

 Executor

(update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)

 ParameterHandler

(getParameterObject, setParameters)

 StatementHandler

(prepare, parameterize, batch, update, query)

 ResultSetHandler

(handleResultSets, handleOutputParameters)

从这四个接口名可以看出它们的作用，其中Executor是执行的接口，ParameterHandler是参数设置接口，StatementHandler是处理Sql语句的接口，ResultSetHandler是处理结果集的接口。

# Mybatis分页/分批

分页是算是我们项目中经常出现的功能，它能防止在大数据量的情况下导致数据库服务器压力剧增，可以减轻每次请求传输数据量消耗的网络流量。在我看来，用户极少会需要看到所有的数据，基本上大部分需求都是看第一页的数据，不信你想想你百度时你会有耐心看10页的数据量吗?哈哈，扯多了。


我的分页需求：

1. 
前端传来分页信息（页码pageSize，每页最大数据量pageNum），后端返回的数据不超过pageNum。

1. 
后端sql不需要自己分页sql，只要在原来的sql上框架自动增加分页sql信息。


思路：

1. 
拦截Mybatis的处理Sql语句的地方，在该地方增加sql分页信息。


1. 
针对不同数据库可以扩展，对外暴露一个统一接口。


首先想到的是拦截StatementHandler接口的prepared方法，拦截后修改sql分页信息。这样处理确实能搞定，但是这里只是实现了分批功能，分页功能是能知道当前所有的记录数，这样我自己还需要写count汇总sql，并且手工拼凑Page分页实例。

按照如上方式，基本上已经实现了分页。由于作者是个代码洁癖男，或者说程序员是懒惰性的，重复的事情可以做一两遍，但绝对不愿意出现第三遍、第四遍。。。就这样，又继续了我的改造之路，我开始在想我的count语句是不是也可以由自己的框架来实现，最好是连拼装分页信息bean也一并帮忙做了，这样开发起来是不是更嗨。。。

经过一番折腾，我找到了Executor接口的query方法，这个接口是先于其他三个接口执行，也就是说我应该可以在这里处理分页sql，并组装count汇总sql，并且该方法的结果集返回的是最终List结果集。以下是我的最后思路：

1. 
拦截query方法，获取方法的MappedStatement(可以说是这个类就是Mybatis的精华)参数实例。

1. 
复制MappedStatement参数实例，组装一个分页sql的MappedStatement实例。

1. 
再组装一个count汇总sql的MappedStatement实例。

1. 
最后执行这两个实例，得到结果List和count数量，将它们设入到PageList实例中，最后返回。


总结：在这里还需要了解Mybatis的MappedStatement、BoundSql、SqlSource这三个类，当然Configuration、SqlSessionFactory也不可少，后面有时间在讨论这几个类吧![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)


