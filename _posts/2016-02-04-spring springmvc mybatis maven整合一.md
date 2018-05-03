---
layout: post
title: [spring springmvc mybatis maven整合一]
categories: [Spring,Mybatis,SpringMVC,Java]
tags: [spring,springmvc,mybatis,maven整合]
id: [18599792082944]
fullview: false
---
# 1.概述

maven项目目录如下

![](http://file.ctosb.com/upload/image/20170705/1499239756426023829.png)

src/main/java：主要java文件路径

src/main/resources：主要资源文件路径

src/test/java：java测试文件目录

src/test/resources：java测试资源文件目录

com.cygoat.controller：存放spring mvc controller.java文件

com.cygoat.dao：存放mybaitis Mapper.java文件

com.cygoat.mapper：存放mybaitis Mapper.xml文件

com.cygoat.model：存放Model.java文件

com.cygoat.service：存放业务层接口类Manager.java文件

com.cygoat.service.impl：存放业务层实现类ManagerImpl.java文件

# 2.maven web项目创建

1.新建项目SSM

![](http://file.ctosb.com/upload/image/20170705/1499239767206015623.png)

2.点击下一步

![](http://file.ctosb.com/upload/image/20170705/1499239777849098953.png)

3.点击完成，创建项目成功。

# 3.spring和mybatis整合

1.pom.xml增加依赖jar包
<properties> <spring.version>3.2.13.RELEASE</spring.version> <mybatis.version>3.2.8</mybatis.version> <mybatis-spring.version>1.2.2</mybatis-spring.version> <druid.version>0.2.9</druid.version> <mysql.version>5.1.27</mysql.version> </properties> <dependencies> <!-- spring context --> <dependency> <groupId>org.springframework</groupId> <artifactId>spring-context</artifactId> <version>${spring.version}</version> </dependency> <!-- spring jdbc --> <dependency> <groupId>org.springframework</groupId> <artifactId>spring-jdbc</artifactId> <version>${spring.version}</version> </dependency> <!-- spring transaction --> <dependency> <groupId>org.springframework</groupId> <artifactId>spring-tx</artifactId> <version>${spring.version}</version> </dependency> <!-- aspectj --> <dependency> <groupId>org.aspectj</groupId> <artifactId>aspectjweaver</artifactId> <version>1.6.11</version> </dependency> <!-- mybatis 包 --> <dependency> <groupId>org.mybatis</groupId> <artifactId>mybatis</artifactId> <version>${mybatis.version}</version> </dependency> <!-- mybatis spring 整合包 --> <dependency> <groupId>org.mybatis</groupId> <artifactId>mybatis-spring</artifactId> <version>${mybatis-spring.version}</version> </dependency> <!-- mysql 驱动包 --> <dependency> <groupId>mysql</groupId> <artifactId>mysql-connector-java</artifactId> <version>${mysql.version}</version> <scope>runtime</scope> </dependency> <!-- 阿里巴巴数据库连接池 --> <dependency> <groupId>com.alibaba</groupId> <artifactId>druid</artifactId> <version>${druid.version}</version> <scope>runtime</scope> </dependency> <dependency> <groupId>junit</groupId> <artifactId>junit</artifactId> <version>4.8.2</version> <scope>test</scope> </dependency> </dependencies>

2.配置文件内容如下

在src/main/resources创建spring.xml文件，内容如下
<?xml version="1.0" encoding="UTF-8"?> <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx" xmlns:aop="http://www.springframework.org/schema/aop" xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd "> <!-- 配置数据源 --> <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> <!-- 基本属性 url、user、password --> <property name="url" value="${url}" /> <property name="username" value="${username}" /> <property name="password" value="${password}" /> <!-- 配置初始化大小、最小、最大 --> <property name="initialSize" value="1" /> <property name="minIdle" value="1" /> <property name="maxActive" value="20" /> <!-- 配置获取连接等待超时的时间 --> <property name="maxWait" value="60000" /> <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 --> <property name="timeBetweenEvictionRunsMillis" value="60000" /> <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 --> <property name="minEvictableIdleTimeMillis" value="300000" /> <property name="validationQuery" value="SELECT 'x'" /> <property name="testWhileIdle" value="true" /> <property name="testOnBorrow" value="false" /> <property name="testOnReturn" value="false" /> <!-- 打开PSCache，并且指定每个连接上PSCache的大小 --> <property name="poolPreparedStatements" value="true" /> <property name="maxPoolPreparedStatementPerConnectionSize" value="20" /> <!-- 配置监控统计拦截的filters，去掉后监控界面sql无法统计 --> <property name="filters" value="stat" /> </bean> <!-- spring和MyBatis完美整合，不需要mybatis的配置映射文件 --> <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean"> <property name="dataSource" ref="dataSource" /> <!-- 自动扫描mapping.xml文件 --> <property name="mapperLocations" value="classpath:com/cygoat/mapper//*.xml"></property> </bean> <!-- DAO接口所在包名，Spring会自动查找其下的类 --> <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer"> <property name="basePackage" value="com.cygoat.dao" /> <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property> </bean> <!-- (事务管理)transaction manager, use JtaTransactionManager for global tx --> <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> <property name="dataSource" ref="dataSource" /> </bean> </beans>

jdbc连接
hibernate.dialect=org.hibernate.dialect.MySQLDialect driverClassName=com.mysql.jdbc.Driver validationQuery=SELECT 1 url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8 username=root password=lxy123 hibernate.hbm2ddl.auto=update hibernate.show_sql=true hibernate.format_sql=true

# 4.maven集成mybatis代码生成器

1.在pom中增加mybatis插件
<plugin> <groupId>org.mybatis.generator</groupId> <artifactId>mybatis-generator-maven-plugin</artifactId> <version>1.3.2</version> <executions> <execution> <id>Generate MyBatis Artifacts</id> <goals> <goal>generate</goal> </goals> </execution> </executions> <configuration> <verbose>true</verbose> <overwrite>true</overwrite> <!-- <jdbcDriver>com.mysql.jdbc.Driver</jdbcDriver> --> <!-- <jdbcURL>jdbc:mysql://127.0.0.1:3306/test</jdbcURL> --> <!-- <jdbcUserId>test</jdbcUserId> --> <!-- <jdbcPassword>test</jdbcPassword> --> </configuration> <dependencies> <dependency> <groupId>mysql</groupId> <artifactId>mysql-connector-java</artifactId> <version>5.1.6</version> </dependency> <dependency> <groupId>org.mybatis.generator</groupId> <artifactId>mybatis-generator-core</artifactId> <version>1.3.2</version> </dependency> <dependency> <groupId>org.mybatis</groupId> <artifactId>mybatis</artifactId> <version>3.2.8</version> </dependency> </dependencies> </plugin>

2.在src/main/resources下创建generatorConfig.xml，文件内容如下
<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd"> <generatorConfiguration> <!-- 数据库驱动--> <context id="DB2Tables" targetRuntime="MyBatis3"> <commentGenerator> <property name="suppressDate" value="true"/> <!-- 是否去除自动生成的注释 true：是 ： false:否 --> <property name="suppressAllComments" value="true"/> </commentGenerator> <!--数据库链接URL，用户名、密码 --> <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:3306/test" userId="root" password="lxy123"> </jdbcConnection> <javaTypeResolver> <property name="forceBigDecimals" value="false"/> </javaTypeResolver> <!-- 生成模型的包名和位置--> <javaModelGenerator targetPackage="com.cygoat.model" targetProject="src/main/java"> <property name="enableSubPackages" value="true"/> <property name="trimStrings" value="true"/> </javaModelGenerator> <!-- 生成映射文件的包名和位置--> <sqlMapGenerator targetPackage="com.cygoat.mapper" targetProject="src/main/java"> <property name="enableSubPackages" value="true"/> </sqlMapGenerator> <!-- 生成DAO的包名和位置--> <javaClientGenerator type="XMLMAPPER" targetPackage="com.cygoat.dao" targetProject="src/main/java"> <property name="enableSubPackages" value="true"/> </javaClientGenerator> <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名--> <table tableName="sys_user" domainObjectName="SysUser" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table> </context> </generatorConfiguration>

3.执行命令生成代码mybatis-generator:generate

![](http://file.ctosb.com/upload/image/20170705/1499239804445020204.png)

此时已将spring，mybatis和mybatis插件整合起来，接下来将要整个spring和springmvc。待续中。。。[spring springmvc mybatis maven整合二](http://ctosb.com/article/18602787082944)
