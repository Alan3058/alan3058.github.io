---
layout: post
title: [Spring Security学习三——自定义用户权限数据源、用户授权]
categories: [Spring]
tags: [spring security,权限数据源,用户授权]
id: [18775952850944]
fullview: false
---

# 背景

在上个例子中，通过将用户资源权限等信息配置在配置文件中，就实现了一个简单的登录、授权功能。然而实际情况我们总是喜欢将用户资源权限得信息保存到数据库、文件或者其他媒介。现在我想将用户权限信息保存到数据库里该怎么办呢？

# 开始


按照官方文档指示，我们应该需要增加一个过滤器FilterSecurityInterceptor，实现数据源接口FilterInvocationSecurityMetadataSource，实现获取用户信息接口UserDetailsService，实现授权管理接口AccessDecisionManager。

本人比较发懒，暂时不想将用户权限信息存放数据库中，而是存放在内存中，不过道理是一样![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)。如下代码开始搞起，附有部分注释。

新增UserDetailsService类，该类的重要作用是提供用户信息。


```java
package com.ctosb.security;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

/**
 * 获取UserDetails的服务类
 * 
 * @author Alan
 * @time 2016年6月3日 上午12:30:12
 *
 */
public class UserDetailsService implements org.springframework.security.core.userdetails.UserDetailsService {

	// 用户和权限信息，可替换成数据库
	static Map<String, UserDetails> map = new HashMap<String, UserDetails>();
	static {
		loadUser("admin", "admin", "ROLE_ADMIN", "ROLE_USER");
		loadUser("alan", "alan", "ROLE_USER");
	}

	public static void loadUser(String username, String password, String... auths) {
		List<GrantedAuthority> grantedAuthorities = new ArrayList<GrantedAuthority>();
		for (String auth : auths) {
			grantedAuthorities.add(new SimpleGrantedAuthority(auth));
		}
		map.put(username, new User(username, password, true, true, true, true, grantedAuthorities));

	}

	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		return map.get(username);
	}

}
```

新建过滤器类数据源FilterInvocationSecurityMetadataSource，该类的作用是提供资源权限信息。

```java
package com.ctosb.security;

import java.util.ArrayList;
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

import org.springframework.security.access.ConfigAttribute;
import org.springframework.security.access.SecurityConfig;
import org.springframework.security.web.FilterInvocation;

/**
 * 过滤器数据源，可认为是资源权限映射关系
 * 
 * @author Alan
 * @time 2016年6月3日 上午12:29:06
 *
 */
public class FilterInvocationSecurityMetadataSource
		implements org.springframework.security.web.access.intercept.FilterInvocationSecurityMetadataSource {

	// 相当于数据库的页面和权限，这里可替换成数据库
	static Map<String, Collection<ConfigAttribute>> map = new HashMap<String, Collection<ConfigAttribute>>();
	static {
		loadResourceAuth("/admin.jsp", "ROLE_ADMIN");
		loadResourceAuth("/index.jsp", "ROLE_USER");
	}

	public static void loadResourceAuth(String url, String... auths) {
		Collection<ConfigAttribute> configAttributes = new ArrayList<ConfigAttribute>();
		for (String auth : auths) {
			configAttributes.add(new SecurityConfig(auth));
		}
		map.put(url, configAttributes);
	}

	@Override
	public Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException {
		String url = ((FilterInvocation) object).getRequestUrl();
		return map.get(url);
	}

	@Override
	public Collection<ConfigAttribute> getAllConfigAttributes() {
		return null;
	}

	@Override
	public boolean supports(Class<?> clazz) {
		return true;
	}

}
```

新建授权服务类AccessDecisionManager，该类主要是给用户授权使用

```java
package com.ctosb.security;

import java.util.Collection;

import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.access.ConfigAttribute;
import org.springframework.security.authentication.InsufficientAuthenticationException;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;

/**
 * 主要用来做访问授权使用
 * 
 * @author Alan
 * @time 2016年6月3日 上午12:26:58
 *
 */
public class AccessDecisionManager implements org.springframework.security.access.AccessDecisionManager {

	@Override
	public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)
			throws AccessDeniedException, InsufficientAuthenticationException {
		for (ConfigAttribute configAttribute : configAttributes) {
			for (GrantedAuthority grantedAuthority : authentication.getAuthorities()) {
				if (grantedAuthority.getAuthority().equals(configAttribute.getAttribute())) {
					return;
				}
			}
		}
		throw new AccessDeniedException("403,access denied");
	}

	@Override
	public boolean supports(ConfigAttribute attribute) {
		return true;
	}

	@Override
	public boolean supports(Class<?> clazz) {
		return true;
	}

}
```

在这里我并没有实现自定义过滤器，而是使用spring原来的过滤器，只是在配置文件中给这个过滤器注入了自己定义的授权管理类AccessDecisionManager属性。spring -security.xml配置文件修改内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/security
        http://www.springframework.org/schema/security/spring-security.xsd">
	<debug/>
	<!-- 自动配置模式，设置访问受限Url-->
	<http auto-config="true" access-denied-page="/403.jsp">
		<intercept-url pattern="/login.jsp" access="IS_AUTHENTICATED_ANONYMOUSLY" />
		<!-- <intercept-url pattern="/index.jsp" access="ROLE_USER" />
		<intercept-url pattern="/admin.jsp" access="ROLE_ADMIN" /> -->
		<!-- 设置登录页面，设置登录失败页面 -->
		<form-login login-page="/login.jsp" login-processing-url="/login"
			authentication-failure-url="/login.jsp?login_error=1" />
		<!-- 设置登出的url，设置登出成功后跳转的页面 -->
		<logout logout-url="/logout" logout-success-url="/login.jsp" />
		<custom-filter ref="filterSecurityInterceptor" before="FILTER_SECURITY_INTERCEPTOR"/>
	</http>
	<!-- 认证管理器,用户名密码都集成在配置文件中 -->
	<authentication-manager alias="authenticationManager">
		<authentication-provider user-service-ref="userDetailsService">
			<!-- <user-service>
				<user name="admin" password="admin" authorities="ROLE_USER,ROLE_ADMIN" />
				<user name="alan" password="alan" authorities="ROLE_USER" />
			</user-service> -->
		</authentication-provider>
	</authentication-manager>
	
	<beans:bean id="filterSecurityInterceptor" class="org.springframework.security.web.access.intercept.FilterSecurityInterceptor">
		<beans:property name="securityMetadataSource" ref="filterInvocationSecurityMetadataSource"></beans:property>
		<beans:property name="authenticationManager" ref="authenticationManager"></beans:property>
		<beans:property name="accessDecisionManager">
			<beans:bean class="com.ctosb.security.AccessDecisionManager"></beans:bean>
		</beans:property>
	</beans:bean>
	
	<beans:bean id="filterInvocationSecurityMetadataSource" class="com.ctosb.security.FilterInvocationSecurityMetadataSource"></beans:bean>
	<beans:bean id="userDetailsService" class="com.ctosb.security.UserDetailsService"></beans:bean>
</beans:beans>
```

之后启动服务，按照上篇测试，结果和上篇测试结果一致。即访问admin.jsp页面需要ROLE_ADMIN权限才行，否则进入403.jsp页面。

# 总结

如上，实现了自定义数据源和授权功能。如果想要修改成数据库方式存储用户权限信息。只需要修改UserDetailsService类和FilterInvocationSecurityMetadataSource类的实现方式为从数据库中获取即可。![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)![](http://img.baidu.com/hi/jx2/j_0028.gif)

