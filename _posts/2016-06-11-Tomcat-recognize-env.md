---
layout: post
title: [Tomcat启动时增加参数区分开发环境和生产环境]
categories: [devtool]
tags: [tomcat,区分不同环境,增加参数]
id: [18788535762944]
fullview: false
---
在使用tomcat启动项目时，需要在启动时增加参数，比如在使用spring时，我们会通过一个参数来区分生产环境和开发环境。比如现在增加spring.profiles.active=dev参数来区分，可在web.xml中增加如下信息，即当前项目环境一切换到开发相关信息。
```xml
<context-param>
    <param-name>spring.profiles.active</param-name>
    <param-value>dev</param-value>
</context-param>
```
PS:应该还有更简单的方法吧，比如在启动时的命令增加命令行参数，或者修改tomcat启动文件的参数，由于目前还没找到，暂时先这样处理咯!
