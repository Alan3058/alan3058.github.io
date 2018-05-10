---
layout: post
title: [Spring Security学习一——简单应用]
categories: [Spring]
tags: [spring security,简单例子]
id: [18767564242944]
fullview: false
---

# 背景


由于工作需要，项目中使用了Spring Security安全框架，为了更上时代进步，先搭个简单应用玩玩。现学现卖，做个笔记，以便能留下更深映像。

开发工具：Eclipse、Maven3、Jetty6集成

开发语言和框架：Java8、Spring Security3.2.8

# 开始


* 首先使用Eclipse构建一个maven web项目，这个不多讲，so easy，不会的自己脑补去。


* 然后是修改web.xml配置文件，内容如下


```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
	<display-name>Archetype Created Web Application</display-name>
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			classpath:spring*.xml
		</param-value>
	</context-param>
	<filter>
		<filter-name>springSecurityFilterChain</filter-name>
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	</filter>
	<!-- spring security 的过滤器，这里配置拦截所有url -->
	<filter-mapping>
		<filter-name>springSecurityFilterChain</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	<!-- spring web监听器 -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>
</web-app>
```

* 配置spring-security.xml,内容如下



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/security
        http://www.springframework.org/schema/security/spring-security.xsd">
	<!-- 自动配置模式，拦截所有请求，有ROLE_USER才可以通过 -->
	<http auto-config="true">
		<intercept-url pattern="/**" access="ROLE_USER" />
	</http>
	<!-- 认证管理器,用户名密码都集成在配置文件中 -->
	<authentication-manager>
		<authentication-provider>
			<user-service>
				<user name="admin" password="admin" authorities="ROLE_USER" />
			</user-service>
		</authentication-provider>
	</authentication-manager>
</beans:beans>
```

* 配置maven文件，主要是增加spring security坐标，这里它会自动依赖spring web等相关jar包，pom.xml内容如下


```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.ctosb</groupId>
	<artifactId>ctosb-spring-security</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>ctosb_spring_security Maven Webapp</name>
	<url>http://maven.apache.org</url>
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<spring.security.version>3.2.8.RELEASE</spring.security.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>
		<!-- spring security core -->
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-core</artifactId>
			<version>${spring.security.version}</version>
		</dependency>
		<!-- spring security web -->
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-web</artifactId>
			<version>${spring.security.version}</version>
		</dependency>
		<!-- spring security config -->
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-config</artifactId>
			<version>${spring.security.version}</version>
		</dependency>
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.1.3</version>
		</dependency>
	</dependencies>
	<build>
		<finalName>ctosb-spring-security</finalName>
		<plugins>
			<plugin>
				<groupId>org.mortbay.jetty</groupId>
				<artifactId>maven-jetty-plugin</artifactId>
				<version>6.1.26</version>
				<configuration>
					<webDefaultXml>src/main/webapp/WEB-INF/webdefault.xml</webDefaultXml>
					<scanIntervalSeconds>10</scanIntervalSeconds>
					<stopKey>foo</stopKey>
					<stopPort>9999</stopPort>
				</configuration>
				<executions>
					<execution>
						<id>start-jetty</id>
						<phase>pre-integration-test</phase>
						<goals>
							<goal>run</goal>
						</goals>
						<configuration>
							<scanIntervalSeconds>0</scanIntervalSeconds>
							<daemon>true</daemon>
						</configuration>
					</execution>
					<execution>
						<id>stop-jetty</id>
						<phase>post-integration-test</phase>
						<goals>
							<goal>stop</goal>
						</goals>
					</execution>
				</executions>
			</plugin>

			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.3</version>
				<configuration>
					<source>1.6</source>
					<target>1.6</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

如上配置文件pom.xml，在这里我配置了jetty插件，所以需要额外增加一个配置文件webdefault.xml，如果不想用jetty插件，本地有tomcat等容器可以启动亦可。这里就不贴webdefault.xml文件的内容，可以网上搜索。

由于添加了jetty插件，我这里就直接运行 clean install jetty:run可以运行工程，使用本地tomcat等容器的可以按自己方式去启动，关于jetty容器插件大家自己脑补去。

* 经过上面，已经启动成功了，输入url访问工程[http://localhost:8080/ctosb-spring-security/](http://localhost:8080/ctosb-spring-security/)，将出现如下登录页面，这个页面是spring security自带的默认登录页面。 



![blob.png](/ueditor/php/upload/image/20160531/1464706774167448.png "1464706774167448.png")

* 输入用户名admin，密码admin，将会登录访问index.jsp。用户密码如果输入错误，将会提示错误信息。


![blob.png](/ueditor/php/upload/image/20160531/1464706954656086.png "1464706954656086.png")

以上就是一个最简单的spring security应用，so easy吧。

# 总结


* 配置web.xml文件。这里面配置最主要的是配置spring security的过滤器DelegatingFilterProxy，该过滤器是一个是一个spring security过滤拦截的入口。并且配置spring web的监听器ContextLoaderListener,这个是spring web应用必配的。

* 配置spring-security.xml文件。该文件主要配置要访问的url需要什么要的权限，并且这里把用户配置在配置文件里面，没有用数据库，纯属方便测试。

* 配置pom.xml文件。加入spring security相关jar包的坐标



未完待续。。。


