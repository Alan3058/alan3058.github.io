---
layout: post
title: [Tomcat去除项目名称访问项目]
categories: [devtool]
tags: [tomcat,根目录访问,去除项目名称]
id: [18784341458944]
fullview: false
---
在使用Tomcat的过程中，一般都是通过localhost:8080/testProjectName访问。由于工作需要现在要去除项目名称，希望通过localhost:8080就可以访问该项目。经过一番资料的搜索，解决方法如下。在tomcat下的server.xml配置文件中的<Host>节点下加入如下<Context>节点信息。配置如下
```xml
      <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
		<Context   path="/"   docBase="testProjectName" reloadable="true"/> 
      </Host>
```

PS：path指访问的路径，docBase指项目名称，如果项目不再webapps下，则需要配置全路径。
