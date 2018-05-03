---
layout: post
title: [springboot-1.第一个demo程序]
categories: [Spring]
tags: [spring,springboot,gradle,demo程序,案例]
id: [20750203753594880]
fullview: false
---
### 介绍

关于springboot介绍，大家可以去spring官网或者百度上查找相关资料，这两年已经火得一塌糊涂了。这里我先通过创建一个简单的springboot项目，以便了解springboot是如何简化开发流程。

相对于maven的XML繁琐配置，我更偏向于gradle的简洁脚本，因此在这里选择的是创建gradle项目。

### 步骤

以下是创建springboot项目的主要步骤

1.创建一个gradle项目（略）

2.添加springboot jar包

在build.gradle文件中添加springboot相关jar包坐标，build.gradle脚本文件内容如下。

```bash
plugins {
    id 'org.springframework.boot' version '1.5.2.RELEASE'
    id 'java'
}


repositories {
	maven{
    	url "http://maven.aliyun.com/nexus/content/groups/public"
    }
    jcenter()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

3.新增Example.java文件，内容如下

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```

4.运行springboot项目

打开命令行程序，进入springboot项目根目录，执行gradle bootrun命令启动项目。看到类似的输出表示启动成功。


```bash
2017-08-30 19:01:41.668  INFO 8992 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-08-30 19:01:41.670  INFO 8992 --- [           main] Example                                  : Started Example in 1.73 seconds (JVM running for 2.003)
```

5.浏览器验证


浏览器输入[http://localhost:8080](http://localhost:8080)，将会显示"Hello World！"字样，表示验证成功。

源代码地址：[https://github.com/Alan3058/springboot-demo](https://github.com/Alan3058/springboot-demo) commit号：5415f1f16a6e6526d968771e689e9cc4d8933dd3

