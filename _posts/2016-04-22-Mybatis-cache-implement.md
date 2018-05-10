---
layout: post
title: [Mybatis缓存实现]
categories: [Mybatis]
tags: [mybatis,缓存实现]
id: [18742398418944]
fullview: false
---

缓存，第一次接触是在2012年大学看马士兵还是韩顺平的Hibernate的视频的时候。当时听的懵懵懂懂的，但感觉非常高端的样子，估计也是从那时开始就将它定为高端大气上档次的玩意，然后每次面试被问道这玩意的时候就只能呵呵了。。。

最近抱着学习沟通、学习当前新技术的心态去面试了几家公司，发现大伙都在谈论的服务化、微服务、高并发、缓存。通常来说，一般的开发人员在一般开发过程是不会涉及到这些玩意，所以明明知道面试官在扯犊子，但自己却没法答出来，心塞ing。。。回家思考良久，感觉自己还是得静下心来，去重新理清这些高端的问题！！！

今天前任大哥打电话过来，说是我之前的问题已经有思路了，并通知我说需要用缓存机制来实现（老大已经离职创业了去了，还能想到我之前的问题，并且还找了外援处理，点赞，估计这也是大哥牛逼之处吧）![](http://img.baidu.com/hi/jx2/j_0003.gif)![](http://img.baidu.com/hi/jx2/j_0003.gif)![](http://img.baidu.com/hi/jx2/j_0003.gif)。顿时心情激动，这不是我这几天天天被为难的一个东西吗？这次一定要把他理清楚。

之后在网上的找了一篇关于Mybatis和Redis的整合实现，发现挺简单的配置下就好了，然而需要使用Redis服务器，自己又不想去搭建这个东东（即使自己有虚拟机），嫌麻烦。于是就研究起大师的实现思路。这是当时参考的地址[http://blog.csdn.net/fhx007/article/details/12680875](http://blog.csdn.net/fhx007/article/details/12680875)。

经研究发现该类实现了Mybatis的Cache接口。该类中主要创建Redis实例，然后通过该实例将查询出来的数据缓存到Redis服务器上，需要获取的时候直接从Redis服务器上拿即可。

这时一个贱贱的想法在我脑海里萌生出来，我是不是可以直接缓存在本地哈。Redis只是一个缓存的数据源，我完全可以用很多看起来很牛逼的方式来替代他，不就是将数据暂存到一边吗？思路渐渐地就理清了许多，其实只需要一个Map，将需要缓存的数据保存到该Map中即可![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)。

```java
package com.sinoservices.sss.utils;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

import org.apache.ibatis.cache.Cache;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * 使用第三方缓存服务器，处理二级缓存
 * @author Alan
 *
 */
public class MybatisRedisCache implements Cache {

    private static final Logger logger = LoggerFactory.getLogger(MybatisRedisCache.class);

    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private String id;
    private Map<Object, Object> map = new HashMap<Object, Object>();

    public MybatisRedisCache(final String id) {
        if (id == null) {
            throw new IllegalArgumentException("Cache instances require an ID");
        }
        logger.info(">>>>>>>>>>>>>>>>>>>>>>>>MybatisRedisCache:id=" + id);
        this.id = id;
    }

    @Override
    public String getId() {
        logger.info(">>>>>>>>>>>>>" + id);
        return this.id;
    }

    @Override
    public int getSize() {
        return map.size();
    }

    @Override
    public void putObject(Object key, Object value) {
        logger.info(">>>>>>>>>>>>>>>>>>>>>>>>putObject:" + key + "=" + value);
        map.put(key, value);
        // redisClient.set(SerializeUtil.serialize(key.toString()), SerializeUtil.serialize(value));
    }

    @Override
    public Object getObject(Object key) {
        Object value = map.get(key);
        // Object value = SerializeUtil.unserialize(redisClient.get(SerializeUtil.serialize(key.toString())));
        logger.info(">>>>>>>>>>>>>>>>>>>>>>>>getObject:" + key + "=" + value);
        return value;
    }

    @Override
    public Object removeObject(Object key) {
        return map.remove(key);
        // return redisClient.expire(SerializeUtil.serialize(key.toString()), 0);
    }

    @Override
    public void clear() {
        map.clear();
        // redisClient.flushDB();
    }

    @Override
    public ReadWriteLock getReadWriteLock() {
        return readWriteLock;
    }

}
```

经过以上代码调整，ok，搞定，数据库只会进行一次查询，后面的都是从本地缓存Map中获取![](http://img.baidu.com/hi/jx2/j_0038.gif)![](http://img.baidu.com/hi/jx2/j_0038.gif)![](http://img.baidu.com/hi/jx2/j_0038.gif)。

总结：经过这次完结，重新再去看Hibernate或者Mybatis的一级缓存、二级缓存等，似乎瞬间明朗了好多![](http://img.baidu.com/hi/jx2/j_0083.gif)![](http://img.baidu.com/hi/jx2/j_0083.gif)![](http://img.baidu.com/hi/jx2/j_0083.gif)，都是猪鼻子插葱装象。其实你有能力的话，他们都可以是你的菜了![](http://img.baidu.com/hi/jx2/j_0082.gif)![](http://img.baidu.com/hi/jx2/j_0082.gif)![](http://img.baidu.com/hi/jx2/j_0082.gif)。之后再网上又看到了一个类PerpetualCache，这个是Mybatis的默认缓存实现类。仔细膜拜里面的源代码后，才发现它的实现思路和我的基本一致，又一次重复造轮子了，罪恶啊![](http://img.baidu.com/hi/jx2/j_0016.gif)![](http://img.baidu.com/hi/jx2/j_0016.gif)![](http://img.baidu.com/hi/jx2/j_0016.gif)。

后记：其实上面的例子只是实现了大概的内容，还是有很多缺陷的。比如每次CUD操作都会清空所有缓存，不过这些都可以控制，具体实现靠自己咯，目前已有些思路，以后有机会再续吧![](http://img.baidu.com/hi/jx2/j_0019.gif)![](http://img.baidu.com/hi/jx2/j_0019.gif)![](http://img.baidu.com/hi/jx2/j_0019.gif)。具体思路参照[http://www.zyiqibook.com/201504/article0413155800266.html](http://www.zyiqibook.com/201504/article0413155800266.html)，可实现删除当前Mapper的缓存，不影响其他Mapper缓存使用。

