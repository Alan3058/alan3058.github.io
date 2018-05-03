---
layout: post
title: [springboot-5.springboot单元测试]
categories: [Spring]
tags: [spring,springboot,gradle,junit]
fullview: false
---
springboot提供了spring-boot-starter-test jar包去支持单元测试。这个jar包依赖于junit、mockito、spring-test、assertj、hamcrest、jsonpath、jsonassert开发jar包，因此我们的项目不需要再引入这些jar包。

首先我们编写一个测试基类BaseTest，我们需要在该基类上添加两个注解SpringBootTest和RunWith。它们一般都是必要的，其中SpringBootTest注解表示该测试类支持springboot，并且我们需要告诉该注解我们应用的启动类。RunWith表示测试类启动时会自动去发现注解。
package test; import org.junit.runner.RunWith; import org.springframework.boot.test.context.SpringBootTest; import org.springframework.test.context.junit4.SpringRunner; import com.ctosb.springboot.Application; //*/* /* TODO /* @author liliangang-1163 /* @date 2017年9月20日下午12:10:27 /* @see /*/ @RunWith(SpringRunner.class) @SpringBootTest(classes = Application.class) public class BaseTest { }

接下来新增我们的测试类，并继承测试基类，之后就是编写我们的测试用例逻辑代码。

package test; import java.util.List; import org.junit.Assert; import org.junit.Test; import org.springframework.beans.factory.annotation.Autowired; import com.ctosb.springboot.model.User; import com.ctosb.springboot.service.UserService; //*/* /* TODO /* @author liliangang-1163 /* @date 2017年9月20日下午12:10:27 /* @see /*/ public class TestUser extends BaseTest{ @Autowired private UserService userService; @Test public void testGet() { List<User> users = userService.getByName(""); Assert.assertTrue(users!=null); } }

运行该测试用例，将检测测试是否通过。

写测试用例是是一个程序员必备的素养，它能增强你对自己代码的信心。
