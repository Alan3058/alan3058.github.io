---
layout: post
title: [Java gradle web项目改造成Spring boot项目]
categories: [Java]
tags: [java,gradle,改造成springboot项目,springboot,ueditor支持springmvc]
fullview: false
---
首先看我的旧项目内部组成，它是一个gradle项目，包含spring、springmvc、mybatis、jquery、h5、css这些常用的技术，另外还有自定义mybatis分页排序插件，自定义过滤器、拦截器，自定义<点赞>、<加载更多>前端插件，还使用了ueditor编辑器。对于项目转成springboot项目，大概有如下三种方案。

1.springboot直接调用解析原来的xml配置文件

2.springboot已经完美集成了一些框架，并且做到了零xml配置

3.使用spring注解方式和springboot提供的配置接口，将配置通过代码实现。

这里我将上面三种方式混合使用了。针对spring、springmvc，我是直接使用第二种方式集成（springboot自身集成）；而针对自定义mybatis插件、自定义拦截器，使用第一种方式集成（xml配置）；针对自定义过滤器使用第三种方式集成（实现接口方式）。

改造过程：

* 剔除原来的spring相关jar包，加上springboot-starter-web jar包，引入springboot。
* springmvc可以完美整合springboot。删除spring-mvc.xml，将其中自定义拦截器移植到spring.xml中。
* springboot启动类中注入spring.xml文件和spring-mybatis.xml，使得自定义拦截器和自定义mybatis插件安装原先xml方式配置（其实可以通过代码实现方式进行，比如@Bean注解）。
* 自定义过滤器由于在web.xml中配置，故而只能通过@Bean注解方式获取。详见FilterRegistrationBean，该是Filter类的包装类。
* 由于最终springboot是以jar包运行，故而原先的ueditor上传的文件图片，将不能在项目目录中，这里需引入了一个文件服务器，以便所有文件图片都使用文件服务器域名。
* 由于标准的springboot项目只有src/main/java和src/main/resources两个源文件夹，没有单独的web源文件夹，所以我们需要将web源文件夹进行转移。将src/main/webapps下的web资源移动到src/main/resources的public或者static文件夹下（我习惯public放置html,static放置jpg、css、js等文件），如果有类似freemarker模版文件，将它们移动到src/main/resources的templates文件夹下。
* 由于ueditor不支持spring mvc格式，需改造ueditor插件，使其支持spring mvc方式。改造后插件地址[https://github.com/Alan3058/ctosb-ueditor](https://github.com/Alan3058/ctosb-ueditor)
* 引入统一认证模块，将原来的认证拆分出来（还在持续中。。。）。

在改造过程中可以采用迭代的方式实现，每次迭代改造后进行功能测试，提高改造的成功性。以下是将网站改造的部分过程。

### springboot集成spring、springmvc

首先添加集成依赖包，并去除原来的spring和spirngmvc包。
compile("org.springframework.boot:spring-boot-starter-web:1.5.8.RELEASE")

然后添加springboot插件，使项目支持springboot。

apply plugin: 'spring-boot' buildscript { repositories { mavenLocal() } dependencies { classpath"org.springframework.boot:spring-boot-gradle-plugin:1.5.8.RELEASE" } }

新建Application类作为spring默认启动入口。
@SpringBootApplication(exclude = {PersistenceExceptionTranslationAutoConfiguration.class}) public class Application { public static void main(String[] args) { SpringApplication.run(Application.class, args); } }

这样，基本上整合了spring、springmvc。

### 集成自定义mybatis等插件

由于mybatis原先是通过xml配置完成，因此将mybatis原来的xml方式保留，让springboot去调用。将springmvc中自定义的拦截器配置移到spring.xml中，最终只保留两个xml配置文件：spring.xml和springmvc.xml。然后在Application类上添加ImportResource注解，将两个xml设置进去。
@ImportResource({ "spring.xml","spring-mybatis.xml" }) public class Application {

这里还可以把spring-mybatis.xml中的数据源配置信息放到application.properties中，可以减少一个jdbc.properties文件。甚至可以将spring.xml和spring-mybatis.xml文件中的bean以spring的注解方式（@Bean）实现，达到零配置的效果，当然最好还是要留有一部分设置项作为灵活配置，比如放在application.properties文件中。

### 集成自定义过滤器

由于过滤器原先是配置在web.xml文件中，但在springboot中没有web.xml文件，然后springboot提供了接口给我们去将自定义过滤器注入进来。如下是我通过代码注入过滤器的样例。
@Bean public FilterRegistrationBean sessionFilterRegistration() { FilterRegistrationBean registration = new FilterRegistrationBean(); SessionFilter sessionFilter = new SessionFilter(); sessionFilter.setXxx(xxx); registration.setFilter(sessionFilter); registration.addUrlPatterns(urlPattern); registration.setName("sessionFilter"); registration.setOrder(1); return registration; }

大道至简，思路很重要。思路清晰了，剩余的只是基础的Coding过程。

文字不多，但由于遇到了许多坑，谨以此记录增强映像。
