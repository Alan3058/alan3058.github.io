---
layout: post
title: [h2数据库导出sql脚本]
categories: [DB]
tags: [h2,导出,sql脚本]
id: [36642300511649792]
fullview: false
---

因需要将本地h2数据库移植到其他机器上的h2数据库中，首先想到的是将本地的h2数据库导出成SQL脚本文件方式，然后在其他机器上执行SQL脚本，即可完成数据库移植。

h2.jar包中提供了Main类供我们导出sql脚本文件。如下是执行main方法命令生成SQL脚本文件方式（如果密码不为空，则要加上密码选项-password xxxx）

```bash
java -cp h2-1.4.196.jar org.h2.tools.Script -url jdbc:h2:file:~/.h2/test -user root -script test.zip -options compression zip
```

也可在eclipse中执行Script类的Main函数，入参为-url jdbc:h2:file:~/.h2/test -user root -script test.zip -options compression zip。

如果需要将数据库导出excel,并导入excel，可参考如下文章

[http://blog.csdn.net/colinmok/article/details/35785323](http://blog.csdn.net/colinmok/article/details/35785323) 

