---
layout: post
title: [Mybatis源码分析]
categories: [Mybatis]
tags: [mybatis,源码分析]
id: [18578820562944]
fullview: false
---

# 1、Mybatis工程创建

首先创建web Maven项目。

pom.xml文件主要依赖包如下

```java
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.2.8</version>
</dependency>
<dependency>
	<groupId>log4j</groupId>
	<artifactId>log4j</artifactId>
	<version>1.2.17</version>
</dependency>


<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.36</version>
</dependency>

<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<version>4.12</version>
</dependency>
```

创建mybatis-config.xml文件，内容如下

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties>
		<property name="driver" value="com.mysql.jdbc.Driver"/>
		<property name="url" value="jdbc:mysql://localhost:3306/test"/>
		<property name="username" value="root"/>
		<property name="password" value="lxy123"/>
	</properties>
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${driver}" />
				<property name="url" value="${url}" />
				<property name="username" value="${username}" />
				<property name="password" value="${password}" />
			</dataSource>
		</environment>
	</environments>
	<mappers>
		<mapper resource="org/mybatistest/mapper/BlogMapper.xml" />
	</mappers>
</configuration>
```

创建Blog.java，代码如下

```java
package org.mybatistest.model;

public class Blog {

	private Long id;
	private String name;
	public Long getId() {
		return id;
	}
	public void setId(Long id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	
}
```

创建BlogMapper.java类，代码如下

```java
package org.mybatistest.mapper;

import org.mybatistest.model.Blog;


public interface BlogMapper {

	public Blog selectBlog(long l);
}
```

创建BlogMapper.xml文件，内容如下

```java
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="org.mybatistest.mapper.BlogMapper">  
  <select id="selectBlog" resultType="org.mybatistest.model.Blog">  
    select * from Blog where id = #{id}  
  </select>  
</mapper>
```

在test库中创建blog表，包含id和name字段。

```java
# Global logging configuration
log4j.rootLogger=ERROR, stdout
# MyBatis logging configuration...
log4j.logger.org.mybatistest.mapper=debug
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

创建Junit测试代码，代码如下

```java
	/**
	 * 通过xml文件构建SqlSessionFactory
	 * @throws IOException
	 */
        @Test
	public void testMybatis() throws IOException{
		InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
		SqlSession sqlSession = sqlSessionFactory.openSession();
		BlogMapper blogMapper = sqlSession.getMapper(BlogMapper.class);
		Blog blog = blogMapper.selectBlog(1l);
		System.out.println(blog.getName());
		sqlSession.close();
	}
```

测试结果如下

![](http://file.ctosb.com/upload/image/20170705/1499239652531082474.png)

# 2、源码分析

如上测试案例，可将其分为如下几个步骤。

1.获取Mybatis配置文件的输入流InputStream。

2.通过SqlSessionFactoryBuilder对象去构建SqlSessionFactory实例。

3.通过SqlSessionFactory实例获取一个SqlSession会话实例。

4.SqlSession实例获取对应Mapper实例。

5.Mapper实例进行数据库操作。

6.关闭SqlSession会话，重要，防止资源一直被占用。

## 2.1、获取Mybatis配置文件输入流

Resources类委托ClassLoaderwrapper类去加载配置文件输入流，而ClassLoaderWrapper类加载输入流的实现是通过类加载器ClassLoader去加载文件，最终返回输入流InputStream。

## 2.2、SqlSessionFactoryBuilder构建SqlSessionFactory实例

跟踪SqlSessionFactoryBuilder的build方法代码可知，首先将输入流信息包装成XMLConfigBuilder实例，最终通过该实例解析方法parse返回一个Configuration实例，然后构建一个默认的SqlSessionFactory实例DefaultSqlSessionFactory(Configuration)，并返回。

以下是解析xml文档成Configuration实例步骤。

1.XMLConfigBuilder通过调用XPathParser解析器实例去获取xml文件的configuration根节点root。

2.解析根节点下的属性配置properties节点，并将信息加载到Configuration配置实例中。

3.解析别名节点typeAliases，并通过Configuration实例的TypeAliasRegistry成员变量，调用注册方法将别名注册到TypeAliasRegistry中的Map集合中。

4.解析插件plugins节点，即Mybatis的拦截器，并将拦截器注入到Configuration实例中。

5.解析objectFactory节点，对象创建工厂，可覆盖重写。

6.解析objectWrapperFactory节点，对象包装工厂。

7.解析settings节点，Mybatis的重要配置信息。

8.解析environments节点，Mybatis环境，配置事务和数据源信息。

9.解析databaseIdProvider节点，即支持不同数据库厂商的切换。

10.解析typeHandlers节点，即类型处理器，数据库类型和java类型对应（通过TypeHandlerRegistry注册器类注册）。

11.解析mappers节点，将mapper信息注入到Configuration实例中。在XMLConfigBuilder类中会通过构建XMLMapperBuilder对象去解析mapper.xml文件sql映射配置信息，通过AnnotationMapperBuilder解析Mapper类上注解的Sql。

## 2.3、获取SqlSession会话实例

通过SqlSessionFactory的openSession方法获得一个SqlSession。DefaultSqlSessionFactory的openSession方法实现如下。

1.Configuration中的环境Environment中获取事务工厂TransactionFactory实例对象，通过事务工厂对象创建一个事务实例。

2.将事务实例对象和Configuration对象包装成对应的Executor，分别是BatchExecutor（用于执行批量sql操作）、ReuseExecutor（重复使用Statement执行sql）和SimpleExecutor（简单sql支持）。

3.如果Mybatis需要开启缓存（默认开启），则将对应的Executor实例对象包装成CachingExecutor。

4.将Executor对象包装成DefaultSqlSession实例返回。

## 2.4、获取对应Mapper实例

跟踪DefaultSqlSession类的getMapper（Class）方法可知，该方法首先调用Configuration类的getMapper（Class，SqlSession）方法，接着在Configuration的方法中获取MapperProxyFactory实例，最后让MapperProxyFactory实例去生成Mapper的代理对象。

MapperProxyFactory实际上是通过Java动态代理生成Mapper代理对象，在该类中创建了MapperProxy实例，MapperProxy类实现了Java的InvocationHandler接口，因此执行数据库的操作入口在MapperProxy类的invoke方法。

## 2.5、执行数据库操作

由于Mapper实例是由Java动态代理机制生成，故而Mapper的执行操作方法是由InvocationHandler的invoke方法完成。MapperProxy代理类实现了InvocationHandler接口类，进入MapperProxy的invoke方法，在该方法中首先将Method包装成MapperMethod实例对象，然后该对象执行excute方法。具体执行操作是在MapperMethod类的execute方法。

进入MapperMethod的execute方法，首先会不同命令去调用SqlSession类对应的insert、update、delete、select等数据库操作方法。跟踪DefaultSqlSession的数据库操作方法代码实现，可知SqlSession接下来主要做了两件事，第一是从Configuration中去获取映射的sql语句包装类MappedStatement，通过Executor去执行MappedStatement的sql语句，封装返回的结果集。

1.获取映射的sql语句包装类MappedStatement。

在Mybatis初始化解析Mapper.xml配置文件时会将sql语句和对应标识保存在一个Map集合中，并设入Configuration实例的成员变量中，因此获取MappedStatement实例，只需从Configuration对象中去获取。

2.Executor对象执行MappedStatement的sql语句。

CachingExecutor，该类带有缓存功能，Mybatis默认是开启缓存功能。CachingExecutor内部有一个Excutor实现类，CachingExecutor对数据库的操作功能都是委托Executor的子类实现。Executor的实际子类有三种BatchExecutor（用于执行批量sql操作）、ReuseExecutor（重复使用Statement执行sql）和SimpleExecutor（简单sql支持），这三个类的数据库实现方式。

现在来看SimpleExecutor实现方式，举一个简单的sql查询例子，首先SqlSession会调用Executor的query方法，该方法在SimpleExecutor的父类BaseExecutor实现，该方法调用子类SimpleExecutor重写的doQuery方法去查询（这里用到模版模式）。

进入doQuery方法可知，该方法通过创建一个RoutingStatementHandler对象来执行Statement的sql语句。实际上RoutingStatementHandler是委托StatementHandler的子类来完成sql的执行。StatementHandler的实现子类有三个SimpleStatementHandler、PreparedStatementHandler和CallableStatementHandler，这三个类都继承BaseStatementHandler。这三个子类最终都是直接调用Statement的execute方法执行sql并返回结果。

## 2.6、SqlSession会话关闭

关闭会话，释放资源。

# 3、结语

最后，梳理一下Mybatis的运行过程。首先获取Mybatis配置文件的输入流，然后通过SqlSessionFactoryBuilder对象构建一个SqlSessionFactory实例，在构建过程中，会对配置文件进行解析，并将解析的相关信息注入到Configuration中，同时当配置了mapper信息时，会从mapper xml文件配置和对应java文件的注解查找sql语句的映射信息，并将映射信息注入到Configuration。之后从SqlSessionFactory中获取SqlSession实例，该实例对象由事务工厂和Excutor执行器构造，即给sql执行期间增加事务管理。之后就从SqlSession中获取需要操作的Mapper实例，并执行相应的方法获取对应结果，该Mapper实例是通过Java动态代理生成。最后关闭SqlSession，释放资源。


