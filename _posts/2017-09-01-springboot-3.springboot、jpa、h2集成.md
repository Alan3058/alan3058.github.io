---
layout: post
title: [springboot-3.springboot、jpa、h2集成]
categories: [Spring]
tags: [spring,springboot,gradle,jpa,h2,集成]
fullview: false
---
### 介绍

前两章介绍了springboot的注解的使用，并通过创建一个demo项目，了解springboot的简单使用，在本章中将整合springboot、spring data jpa、h2，并实现一个简单的用户操作，来验证整合结果。

注：对spring data jpa和h2不了解的，可自行充电，对这两个东东了解我也是新手学习阶段。

### 步骤

1.首先在build.gradle脚本中添加spring data jpa和h2的依赖jar包坐标。
compile("org.springframework.boot:spring-boot-starter-data-jpa") //compile("org.springframework.boot:spring-boot-devtools") compile("com.h2database:h2:1.4.196")

2.新建User、UserRepository、UserService类。

User类内容如下
@Entity public class User { @Id @GenericGenerator(name = "system-uuid", strategy = "uuid") //@SequenceGenerator(name = "id_generator", sequenceName = "id_sequence", initialValue = 23) @GeneratedValue(generator = "system-uuid") private String id; @Column private String name; @Column private Integer age; public User() { } public User(String name, Integer age) { this.name = name; this.age = age; } public String getId() { return id; } public void setId(String id) { this.id = id; } public String getName() { return name; } public void setName(String name) { this.name = name; } public Integer getAge() { return age; } public void setAge(Integer age) { this.age = age; } }

UserRepository类内容如下

public interface UserRepository extends JpaRepository<User, String>{ List<User> getByNameLike(String userName); }

UserService类内容如下

@Component public class UserService { @Autowired private UserRepository userRepository; public List<User> getByName(String name) { return userRepository.getByNameLike(name); } public User insert(User user) { return userRepository.save(user); } }

最后在ExampleController类中添加新增和查询接口方法。

@RequestMapping(value = "/user", method = RequestMethod.POST) @ResponseBody public Object addUser(@RequestBody User user) { return userService.insert(user); } @RequestMapping(value = "/user", method = RequestMethod.GET) @ResponseBody public Object get(@RequestParam("name") String name) { return userService.getByName(name + "%"); }

3.启动项目并验证。

通过[http://localhost:8080/user](http://localhost:8080/user) post验证添加用户，[http://localhost:8080/user](http://localhost:8080/user) get方法验证查询用户。

### 其他设置项

由于springboot天然集成h2，所以在上面我们都不需要配置数据库连接信息，springboot使用了默认的配置。接下来我们增加点自定义配置，以便调试方便。

1.新增src/main/resources源文件，在resources文件下新建application.properties配置文件。

2.之后添加h2数据库连接信息，我们这里使用h2的持久化本地模式。
/#数据库连接信息(file表示h2持久化本地，mem表示h2内存型数据库） spring.datasource.url=jdbc:h2:file:~/.h2/test spring.datasource.driver-class-name=org.h2.Driver spring.datasource.username=root spring.datasource.password=

3.开启h2 web 控制台管理配置

开启h2 web 控制台管理配置有两种方式，一种是将spring.h2.console.enabled设为true；第二种是在项目中添加spring-devtool jar包，它将自动开启web控制台管理。就可以通过浏览器[http://localhost:8080/h2](http://localhost:8080/h2)去管理h2数据库。
/#开启h2 web控制台管理,使用如下标记或者使用spring-devtool spring.h2.console.enabled=true /#h2 web控制台访问路径，默认为h2-console spring.h2.console.path=/h2

4.添加初始化sql脚本

默认情况下springboot每次启动都会通过hibernate自定义建表功能，将数据库表结构删除，然后重新建表，我这里准备采用自定义sql脚本，即让springboot每次启动都执行我的sql脚本。

首先关闭hibernate自定义建表开关
spring.jpa.generate-ddl=false spring.jpa.hibernate.ddl-auto=none

配置每次建表sql和插入数据sql的位置。

spring.datasource.schema=classpath:sql/schema.sql spring.datasource.data=classpath:sql/data.sql

这样在每次启动时就会去执行我们自定义的初始化sql脚本。

5.添加打印sql日志配置

最初使用了显示执行sql配置
/#显示jpa执行sql spring.jpa.show-sql=true

这种方式不会打印完整的带参数的sql，所以后面我使用了log4jdbc jar包，它可以打印完整的sql。

在buid.gradle脚本添加jar包依赖。
compile("com.googlecode.log4jdbc:log4jdbc:1.2")

修改数据库连接驱动和url

spring.datasource.url=jdbc:log4jdbc:h2:file:~/.h2/test spring.datasource.driver-class-name=net.sf.log4jdbc.DriverSpy

这样日志信息就会输出完整sql信息。

源代码地址：[https://github.com/Alan3058/springboot-demo](https://github.com/Alan3058/springboot-demo) commit号：

7fbb86e437032f52e1991fb98484999f7218edc4
