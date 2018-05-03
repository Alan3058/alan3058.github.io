---
layout: post
title: [Spring Security学习二——自定义登录页面、增加登出功能、增加权限控制]
categories: [Spring]
tags: [spring security,自定义登录页面,登出页面]
fullview: false
---
# 背景

在前面的例子中已经完成了一个最简单的Spring Security的登录例子。现在需要增加如下功能

* 自定义登录页面
* 增加登出功能
* 增加不同用户有不同权限控制
* 增加访问受限，跳到访问首先页面

# 开始

修改spring-security.xml配置文件。
<?xml version="1.0" encoding="UTF-8"?> <beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd"> <debug /> <!-- 自动配置模式，设置访问受限Url--> <http auto-config="true" access-denied-page="/403.jsp"> <intercept-url pattern="/login.jsp" access="IS_AUTHENTICATED_ANONYMOUSLY" /> <intercept-url pattern="/index.jsp" access="ROLE_USER" /> <intercept-url pattern="/admin.jsp" access="ROLE_ADMIN" /> <!-- 设置登录页面，设置登录失败页面 --> <form-login login-page="/login.jsp" authentication-failure-url="/login.jsp?login_error=1" /> <!-- 设置登出的url，设置登出成功后跳转的页面,设置登录处理url --> <logout logout-url="/logout" logout-success-url="/login.jsp" login-processing-url="/login" /> </http> <!-- 认证管理器,用户名密码都集成在配置文件中 --> <authentication-manager> <authentication-provider> <user-service> <user name="admin" password="admin" authorities="ROLE_USER,ROLE_ADMIN" /> <user name="alan" password="alan" authorities="ROLE_USER" /> </user-service> </authentication-provider> </authentication-manager> </beans:beans>

如上配置，增加了新的用户alan，并给admin用户添加ROLE_ADMIN权限。设置登录和登出页面，设置访问受限页面。

登录页面login.jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%> <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%> <html> <body> <c:if test="${not empty param.login_error}"> <font color="red">login fail,please try again.<br /> </font> </c:if> <form action="login" method="post" style="text-align:center"> username: <input type='text' name='j_username'><br> password: <input type='password' name='j_password'><br> <input name="submit" type="submit" value="submit"> <input name="reset" type="reset" value="reset"> </form> </body> </html>

修改index.jsp页面如下
<html> <body> <h2>welcome login!</h2> <a href="admin.jsp">access admin page</a> <a href="logout">logout</a> </body> </html>

新增admin.jsp页面，该页面只有ROLE_ADMIN权限可访问
<html> <body> <h2>welcome admin page!</h2> </body> </html>

新增403.jsp访问受限界面
<html> <body> <h2>403 error,access denied.</h2> <a href="login.jsp">to reLogin</a> </body> </html>

启动web服务器。登录admin用户，可以访问admin.jsp界面。登录alan用户访问admin.jsp将会跳转到403页面。

![blob.png]( "1464796510509145.png")

# 总结

主要的配置信息都是在spring-security.xml文件，该文件可配置页面权限，用户信息等。后续需要将用户信息保存到数据库中。

未完待续。。。
