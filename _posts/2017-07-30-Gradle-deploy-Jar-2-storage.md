---
layout: post
title: [Gradle发布Jar包到公共Maven仓库]
categories: [devtool,Gradle]
tags: [gradle,发布jar包,公共仓库]
id: [16262108207382528]
fullview: false
---
### 步骤

1. 进入sonatype官网（[https://issues.sonatype.org/](https://issues.sonatype.org/ "https://issues.sonatype.org/")）注册帐号，然后申请创建ISSUE清单。参考文章[http://blog.csdn.net/qiujuer/article/details/44195199](http://blog.csdn.net/qiujuer/article/details/44195199 "http://blog.csdn.net/qiujuer/article/details/44195199")。

2. 使用GPGTools，生成密钥对。sonatype规定，要上传jar包到sonatype仓库中，需要对包进行加密签名。可参考文章[http://blog.csdn.net/qiujuer/article/details/44173611](http://blog.csdn.net/qiujuer/article/details/44173611 "http://blog.csdn.net/qiujuer/article/details/44173611")。

3. 之后通过gradle脚本中引入maven和signing插件，然后配置打包和上传任务。这里打包需要注意：sonatype规定必须上传三个jar包，分别是jar、javadoc、sourcesJar。可参考文章[http://blog.csdn.net/qiujuer/article/details/44195131](http://blog.csdn.net/qiujuer/article/details/44195131)，[http://ningandjiao.iteye.com/blog/1846441](http://ningandjiao.iteye.com/blog/1846441 "http://ningandjiao.iteye.com/blog/1846441")

官方文档地址 [http://central.sonatype.org/pages/ossrh-guide.html](http://central.sonatype.org/pages/ossrh-guide.html)

### 遇到的问题

* 执行命令时遇到401未授权问题，sonatype用户名密码错误。
> gradle Return code is: 401, ReasonPhrase: Unauthorized.

* 执行javadoc任务出错。
> Javadoc generation failed. Generated Javadoc options file (useful for troubleshooting): 'E:\other\test\build\tmp\javadoc\javadoc.options'

这个问题纠结了很久，日志打印不详细，查看javadoc.options文件也查不出原因。最后通过eclipse导出javadoc的功能，eclipse控制台才打印详细信息。主要有两个错误，一个是中文注释问题，一个是不能解析的注解。

中文注释问题通过设置javadoc的编码格式为UTF-8得到解决。

注解问题可通过删除不规范的文档注解，或者在gradle的javadoc中设置failOnError属性值为false，它表示忽略错误。如下是javadoc配置
```gradle
   javadoc {
   	    failOnError false
	    options{
	        encoding "UTF-8"
	        charSet 'UTF-8'
	        author true
	        version true
	        links "http://docs.oracle.com/javase/7/docs/api"
	        title 'test document'
	    }
	}
```

* 执行签名任务报错。
> Cannot perform signing task ':signArchives' because it has no configured signatory

由于没有配置私钥位置，所以报错，需配置正确的私钥位置。

> signing.secretKeyRingFile=C:/Users/dell/AppData/Roaming/gnupg/secring.gpg

### 其他说明

ctosb-core.jar包已推送到公共仓库上，gradle坐标如下。
> compile group: 'com.ctosb', name: 'ctosb-core', version: '1.0.0'

可通过如下url访问

[https://mvnrepository.com/artifact/com.ctosb/ctosb-core](https://mvnrepository.com/artifact/com.ctosb/ctosb-core)

[http://maven.aliyun.com/nexus/content/groups/public/com/ctosb/ctosb-core/](http://maven.aliyun.com/nexus/content/groups/public/com/ctosb/ctosb-core/)

[http://search.maven.org//#search%7Cgav%7C1%7Cg%3A%22com.ctosb%22%20AND%20a%3A%22ctosb-core%22](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.ctosb%22%20AND%20a%3A%22ctosb-core%22)
