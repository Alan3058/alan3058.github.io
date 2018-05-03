---
layout: post
title: [springboot-4.springboot整合freemarker、swagger]
categories: [Spring]
tags: [springboot,gradle,freemarker,swagger,整合]
id: [22541128121188352]
fullview: false
---

freemarker是一款开源，且功能强大的Java后台模板语言，通常会使用它替代jsp。swagger也是一个开源的框架，它可以将我们的api整合起来管理调试，极大提高了我们对系统api管控的效率。以上是个人理解，更多详情请移步对应官网或者百度。

下面是springboot整合freemarker和swagger的大体步骤流程。

### springboot整合freemarker

1.首先在build.gradle脚本文件中添加springboot-freemarker jar包。


```bash
compile("org.springframework.boot:spring-boot-starter-freemarker")
```

2.在resources文件夹下新建templates文件夹，该文件夹是springboot-freemarker默认存放freemarker文件的目录。

3.在templates文件夹下新增hello.ftl模板文件，内容如下。

```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
${message} ${name}
</body>
</html>
```

4.在ExampleController中添加hello接口，用于显示将hello.ftl展现成html页面。

```java
	@RequestMapping(value = "/hello", method = RequestMethod.GET)
	public String hello(Map<String,Object> map) {
		map.put("message", "hello world!");
		map.put("name", "alan");
		return "hello";
	}
```

### springboot整合swagger

1.首先在build.gradle脚本文件中添加springfox-swagger jar包。

```bash
	compile("io.springfox:springfox-swagger2:2.7.0")
	compile("io.springfox:springfox-swagger-ui:2.7.0")
```

2.在新建在com.ctosb.springboot包下新增config子包，并在该包下增加SwaggerConfig.java配置类，设置swagger相关配置项。

```java
package com.ctosb.springboot.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * TODO
 * @author liliangang-1163
 * @date 2017年9月1日下午8:22:18
 * @see
 */
@Configuration
@EnableSwagger2 // 启用 Swagger
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .useDefaultResponseMessages(false)
                .select()
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
            .title("spring boot swagger 接口列表")//大标题
            .version("1.0")//版本
            .build();
    }
}
```

3.之后就可以在ExeampleController.java类和方法上添加注解和对应说明。如下是是添加后简单的例子，swagger注解是以api开头。

```java
@Api(tags = "案例接口")
public class ExampleController {
        @RequestMapping(value = "/user", method = RequestMethod.POST)
	@ResponseBody
	@ApiOperation(value = "新增用户接口")
	public Object addUser(@ApiParam(required = true, name = "user", value = "用户信息") @RequestBody User user) {
		return userService.insert(user);
	}
}
```

4.之后启动项目，输入[http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html),即可进入swagger接口列表，找到对应接口输入对应参数，将会返回响应的结果信息。

源代码地址：[https://github.com/Alan3058/springboot-demo](https://github.com/Alan3058/springboot-demo) commit号：1a011cf49a9208116bd45586eec0201728fd55b8


