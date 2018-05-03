---
layout: post
title: [springboot-2.springboot最佳实践]
categories: [Spring]
tags: [spring,springboot,gradle,EnableAutoConfiguration,ComponentScan,Configuration,最佳实践]
fullview: false
---
上一节中已经搭建了一个简单的springboot项目，它也是springboot官方的demo，接下来通过学习springboot官方的提出来几点注意的地方，来改造第一个demo。以下是我汇总的几点，这里就称呼它们为springboot最佳实践吧。

### springboot最佳实践

1.官方建议一个项目建立一个package，不推荐使用默认包，使用默认包可能导致一些 问题（在使用 @ComponentScan, @EntityScan or @SpringBootApplication注解的情况下）， 因为它会扫描所有的jar包。

2.官方建议main类放在其他类上面的根包中， @EnableAutoConfiguration 注解通常放于 main类中，它的意图是告诉springboot，根据您添加的jar依赖项，“猜测”您将如何配置Spring。

3.通常建议在main方法类上加上 @Configuration ，我们不需要在每个类上添加 @Configuration 注解，只需 使用 @Import 注解去导入其他配置类即可。

4.如果一定要使用xml配置，官方建议应以 @Configuration 注解开始，然后在附加 @ImportResource 注解去加载xml配置。

5.使用 @ComponentScan 注解，springboot将可以自动发现所有的spring组件，包括 @Configuration 类。

6.springboot还有另外一个注解 @SpringBootApplication， 它等价于 @Configuration, @EnableAutoConfiguration and @ComponentScan 三个注解的组合。

### demo改造

根据最佳实践说明，我们将会第一个demo程序进行改造。首先新增Java包com.ctosb.springboot，然后将Example.java类，拆分成两个类Application.java和ExampleController，并分别放于com.ctosb.springboot和com.ctosb.springboot.controller包中。

1.Application.java是main函数启动类，内容如下
package com.ctosb.springboot; import org.springframework.boot.SpringApplication; import org.springframework.boot.autoconfigure.EnableAutoConfiguration; import org.springframework.context.annotation.ComponentScan; import org.springframework.context.annotation.Configuration; //*/* /* 官方建议一个项目建立一个package，不推荐使用默认包，使用默认包可能导致一些 问题（在使用 @ComponentScan, @EntityScan /* or @SpringBootApplication注解的情况下）， 因为它会扫描所有的jar包。<br> /* 官方建议main类放在其他类上面的根包中， @EnableAutoConfiguration 注解通常放于 /* main类中，它的意图是告诉springboot，根据您添加的jar依赖项，“猜测”您将如何配置Spring。<br> /* 通常建议在main方法类上加上 @Configuration ，我们不需要在每个类上添加 @Configuration 注解，只需 使用 @Import /* 注解去导入其他配置类即可。<br> /* 如果一定要使用xml配置，官方建议应以 @Configuration 注解开始，然后在附加 @ImportResource 注解去加载xml配置。<br> /* 使用 @ComponentScan 注解，springboot将可以自动发现所有的spring组件，包括 @Configuration 类。<br> /* springboot还有另外一个注解 @SpringBootApplication， /* 它等价于 @Configuration, @EnableAutoConfiguration and @ComponentScan 三个 /* 注解的组合。如下三个注解可直接使用 @SpringBootApplication 代替。<br> /* @author liliangang-1163 /* @date 2017年8月31日下午8:30:07 /*/ @Configuration @EnableAutoConfiguration @ComponentScan public class Application { public static void main(String[] args) throws Exception { SpringApplication.run(Application.class, args); } }

2.ExampleController是springmvc层的Controller类，内容如下
package com.ctosb.springboot.controller; import org.springframework.web.bind.annotation.RequestMapping; import org.springframework.web.bind.annotation.RestController; @RestController public class ExampleController { @RequestMapping("/") String home() { return "Hello World!"; } }

启动应用程序和验证步骤和第一篇一致，这里不再赘述。

源代码地址：[https://github.com/Alan3058/springboot-demo](https://github.com/Alan3058/springboot-demo)commit号：459577659105c1a479c0e4126450c369dfd12e11
