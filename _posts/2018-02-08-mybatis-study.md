---
layout: post
title:  "mybatis study"
categories: [mybatis]
tags: [mybatis]
fullview: false
---
mybatis框架是一个半自动ORM框架，并且它是目前主流框架套件（SSM）之一，相比Hibernate来说，它是非常简单且容易上手。虽说hibernate声称自己不需要写sql，并且可以跨数据库。但是对于程序员的我们来说，多写写sql还是有好处的，而且哪个公司闲着没事随便换数据库。目前工作中使用mybatis的频率比较高，因此打算对mybatis知识进行一次整理。

# 版本
3.4.5

# mybatis问题

1. mybatis xml配置文件处理？  
   xml配置文件解析入口类是XMLConfigBuilder，它只提供一个parse公有方法，parse方法调用parseConfiguration方法，在该方法中会对配置文件的各个元素节点进行解析，最后将值设置到Configuration等实例中。
   mybatis使用jdk xpath方式解析xml配置文件，mybatis入口类是XPathParser。XPathParser内有一个Document对象和XPath解析对象，通过XPath将对应节点解析成Node，然后将Node封装成mybatis的XNode对象。XNode提供了一系列访问xml元素属性、内容、子节点的接口。  
   `XMLConfigBuilder->XPathParser->XNode->Node(W3c node解析)`
2. mapper xml process？  
   ​mapper xml文件解析入口类是XMLMapperBuilder，它同样提供了一个parse公有方法去解析，它解析xml方式和配置文件的解析一样，都是使用XNode访问xml元素。主要解析cache、parameterMap、resultMap、sql、select、insert、update、delete节点。其中select|insert|update|delete都交由XMLStatementBuilder解析，每个节点对应一个MappedStatement实例。MappedStatement表示一个完整的sql块，它包含sql命令类型、sql源、参数类型、结果集映射、缓存等信息。一般实现自定义分页插件时，可以构造一个MappedStatement去查询总记录数。

3. SqlSessionFactory生成？  
   在解析完xml配置文件和mapper文件后，并将对应配置信息设入Configuration对象中。之后直接创建DefaultSqlSessionFactory实例，并将Configuration对象当做实例变量置入，以便后续直接使用。

4. SqlSession生成？  
   SqlSessionFactory作为SqlSession的工厂类，专门用来生成SqlSession实例。工厂类首先从配置文件中获取配置的事务管理器，默认是JdbcTransactionManager，并从事务管理器中获得一个事务，之后将事务包装成Excutor，最后传给DefaulSqlSession类，并实例化。
5. SqlSession commit 和 rollback？  
   SqlSession的commit和rollback操作会先调用Executor的commit和rollback方法。Executor首先会做一些清理工作（清理本地缓存），然后再执行commit或者rollback操作。
6. Transaction process？  
   mybatis提供了两个事务实现，分别是JdbcTransaction和ManagedTransaction。JdbcTransaction是直接使用jdbc的提交和回滚，依赖于从数据源获取的连接来管理事务；ManagedTransaction事务交由容器来管理（比如tomcat、jetty），并且他的的commit和rollback方法都是空实现。
7. plugin？  
   mybatis的插件机制是由jdk的动态代理实现，只提供了四个接口可以被代理：ParameterHandler、StatementHandler、ResultSetHandler和Executor，它们分别是处理参数、处理sql、处理结果集、执行器。即我们可以自定义处理参数逻辑、自定义处理sql、自定义处理执行结果、自定义执行过程。  
   mybatis提供了插件链InterceptorChain类，去存储所有的插件Interceptor。将插件注入到这些接口的入口是Configuration的newParameterHandler、newResultSetHandler、newStatementHandler、newExecutor，它们都调用了InterceptorChain的pluginAll去将插件注入到对应处理实例中。
   mybatis还提供了Plugin类，封装了动态代理目标对象，以便支持插件功能。

8. exception process？  
   mybatis定义了PersistenceException异常基类，并且在每个包（模块）中都定义了对应的异常，它们都是PersistenceException的子类。同时mybatis提供了ExceptionFactory工厂类，该类只有一个公有静态方法wrapException，它的作用是将各种异常包装统一包装成PersistenceException实例。
9. logging process？  
  mybatis提供了统一的Log日志接口，并且支持四种日志级别（error、debug、trace、warn），通过**适配器模式**去适配目前市面上大部分主流的日志框架。其实现原理是在应用启动时去加载各个日志框架的实现类，如果加载成功，则表示使用该日志框架，其加载顺序是`slf4j->commonslogging->log4j2->log4j->jdklog->nolog`。入口类LogFactory。  
  有了日志工厂类，mybatis对jdbc的Connection、Statement、PreparedStatement、ResultSet接口进行动态代理，并为它们增加日志记录功能。详见代理实现类是ConnectionLogger、StatementLogger、PreparedStatementLogger、ResultSetLogger。
10. annotation？  
   在配置了扫描mapperClass类或者package时，mybatis会去扫描对应java类或者package，然后调用XMLAnnotationBuilder去扫描对应的Mapper类并解析。调用入口是XMLConfigBuilder的mapperElement方法，主要通过反射获取注解及注解上的信息。比如Select注解（它的value值就是sql），mybatis会将sql值解析到MappedStatement对象中。
11. cache module？  
   mybatis提供了Cache接口去实现缓存，并且提供了一个默认缓存实现PerpetualCache，它内部使用一个map变量去缓存数据。不仅如此，mybatis还提供了CacheKey去支持缓存key的实现。根据缓存的特性，mybatis还提供了一系列装饰器Cache实现类去加强Cache的功能，这些类都在decorators包中，比如LruCache、FifoCache、BlockingCache等（从名字上可以知道他们的作用）。  
   再有了缓存实现的类的情况下，mybatis支持两个纬度的缓存，一个是SqlSession级别，另一个是SqlSessionFactory级别。分别表示同一个SqlSession下的相同MappedStatement和参数执行多次，实际只会执行一条sql;同一个SqlSessionFactory下的相同MappedStatement和参数执行多次，最终只会执行一条sql。

#  mybatis例子

```java 
// 指定MyBatis配置文件
String configFile = "mybatis-config.xml";
// 1、指定MyBaties配置文件
InputStream inputStream = Resources.getResourceAsStream(configFile);
// 2、创建SqlSessionFactory()
SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession session = null;
try {
	// 3、获取SqlSession
	session = sessionFactory.openSession();
	// 4、获取DAO接口对象
	UserMapper userMapper = session.getMapper(UserMapper.class);
	// 5、CRUD操作
	// 5.1--查询
	User user = userMapper.getUserById(1);
	session.commit();
} catch (Exception e) {
	e.printStackTrace();
} finally {
	// 6、关闭SqlSession
	if (session != null) {
		session.close();
	}
}
```

#  mybatis处理流程
* SqlSessionFactoryBuilder创建SqlSessionFactory实例。
  * SqlSessionFactoryBuilder委派XMLConfigBuilder类去解析mybatis-config.xml文件，并组装成Configuration实例。
    * XMLConfigBuilder将xml文件流委托给XPathParser类去解析成XNode节点。XPathParser通过java自带的jaxp方式将xml解析成Node，然后将Node包装成XNode，方便XMLConfigBuilder解析。
    * XMLConfigBuilder针对XNode节点，解析对应xml节点属性。如plugins节点解析成Interceptor拦截器集合；environments节点解析成Environment对象，等。
  * 将Configuration实例交由DefaultSqlSessionFactory类，并将DefaultSqlSessionFactory实例化返回。
* SqlSessionFactory创建SqlSession实例。
  * 从Configuration中获取Enviroment对象，再从Enviroment对象中获取事务工厂类TransactionFactory对应的实例对象。
  * TransactionFactory事务工厂类创建一个Transaction对象。
  * 通过ExecutorType类型创建与其对应的Executor执行器，该执行器会将事务对象包装起来。默认创建SimpleExecutor执行器。
  * 将Configuration和Executor实例包装成SqlSession对象返回。  
    **可以看出，实际执行数据库操作的是Executor对象，而事务由Transaction控制。**
* SqlSession执行数据库操作（insert、delete、update、select等）。
  * 通过id（mapper.xml文件的命名空间+sql语句的id）查找MappedStatement实例（已将sql包装起来）。
  * 将MappedStatement实例和方法入参交由Executor实例执行，返回结果。  
    **SqlSession执行数据库操作有两种方式，一种是通过mapper.xml的id直接访问，另一种是通过面向Mapper接口方式调用。Mapper接口方式首先会返回一个代理MapperProxy，实际执行方法时，通过接口的完全限定名+方法名称作为id调用SqlSession的方法，即上述步骤。这也就是为什么mapper.xml的命名空间和sql语句id要与Mapper接口的完全限定名和方法名各自对应**
* SqlSession commit/rollback事务。  
  mybatis的事务处理最终依然是调用jdbc Connection的commit和rollback两个方法进行提交和回滚操作。它的调用路线如下：  
  `sqlSession.commit/rollback->executor.commit/rollback->transaction.commit/rollback->connection->commit->rollback`

#  总结
**百度脑图：**[技术学习整理路线](http://naotu.baidu.com/file/2eab9acbf1192229072dbc68eefe641b) mybatis知识块。
