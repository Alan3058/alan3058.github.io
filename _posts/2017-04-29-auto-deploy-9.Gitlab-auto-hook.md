---
layout: post
title: [自动化发布-9.Gitlab每次pull request触发Jenkins构建，检查代码规范和findbug]
categories: [自动化发布]
tags: [自动化发布,代码规范检测,findbug,jenkins,自动]
id: [18901790359552]
fullview: false
---
# 背景

从系统开发到发布过程中，我们可能会有如下几个流程。

1. 开发Leader制作代码规范，要求开发人员按照代码规范去开发。
2. 开发人员提交代码到代码库中，等待开发Leader审核。（以前我们用的是淘宝开发的一款审查工具Tao-ReviewBoard）
3. 开发Leader手工去审查每个开发人员提交上来的代码，是否有bug、或者不符合规范。如果不通过，开发人员重新提交。

为了减少开发Leader人为的去核对每个开发人员的代码所带来繁琐的工作量，我们能否改成自动化的呢，将代码规范和常见bug写到模版中去，让工具自动去检查。答案当然是有的。。。

# 实操

设想：每当开发人员去提交代码到Gitlab上时，就自动去触发Jenkins构建，并检查代码规范，检查代码是否有潜在的bug，单元测试是否通过，然后发送邮件给开发人员通知本次pull request是否通过。

1. Gitlab触发Jenkins自动构建

首先第一个问题是Gitlab是否能够去触发Jenkins的任务，Jenkins需要暴露一个接口给Gitlab去触发。Jenkins的构建触发器中有一栏触发远程脚本，就是暴露出来的接口，而Gitlab的webhook功能是可以在代码被提交时触发一个外部的接口。如下图

![blob.png](http://file.ctosb.com/upload/image/20170429/1493451625372083131.png "1493451625372083131.png")

Jenkins配置图

![blob.png](http://file.ctosb.com/upload/image/20170429/1493451708125058216.png "1493451708125058216.png")

Gitlab配置图

2. Gradle支持检查代码规范、findbug、代码覆盖率、单元测试

在gradle官方支持插件系列中，有支持检查代码规范、查找bug、统计代码覆盖率的插件。在这里我选择了如下几个插件。
```gradle
apply plugin: 'checkstyle' //代码规范
apply plugin: "findbugs" //查找bug
apply plugin: "pmd" //
apply plugin: "jacoco" //代码覆盖率
```

然后在Jenkins中配置check和jacoco任务。
![blob.png](http://file.ctosb.com/upload/image/20170429/1493452593588003599.png "1493452593588003599.png")

详见官方文档[https://docs.gradle.org/3.5/userguide/checkstyle_plugin.html](https://docs.gradle.org/3.5/userguide/checkstyle_plugin.html)

[https://docs.gradle.org/3.5/userguide/findbugs_plugin.html](https://docs.gradle.org/3.5/userguide/findbugs_plugin.html)

[https://docs.gradle.org/3.5/userguide/pmd_plugin.html](https://docs.gradle.org/3.5/userguide/pmd_plugin.html)

[https://docs.gradle.org/3.5/userguide/jacoco_plugin.html](https://docs.gradle.org/3.5/userguide/jacoco_plugin.html)

3. Jenkins检查代码规范性、findbug、代码覆盖率、单元测试

在前面Gradle已经将代码规范、findbug、代码覆盖率结果输出到对应的xml文件中，现在Jenkins需要做的就是将结果收集起来，然后将结果发送给对应的开发人员。如下插件列表  

|插件名称|说明|
|-|-|
|[Checkstyle Plug-in](http://wiki.jenkins-ci.org/x/GYCGAQ)|检查代码规范|
|[FindBugs Plug-in](http://wiki.jenkins-ci.org/x/GYAs)|检查代码bug|
|[JaCoCo plugin](https://wiki.jenkins-ci.org/display/JENKINS/JaCoCo+Plugin)|检查代码覆盖率|
|[Static Analysis Collector Plug-in](http://wiki.jenkins-ci.org/x/tgeIAg)|收集检测结果（包括Checkstyle、Findbugs、Jacoco），统计生成报表|
|[PMD Plug-in](http://wiki.jenkins-ci.org/x/GAAHAQ)| |
|[Email Extension Plugin](http://wiki.jenkins-ci.org/display/JENKINS/Email-ext+plugin)| |
|[Email Extension Template Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Email-ext+Template+Plugin)| 邮箱|

添加完上述插件后，在《构建后操作》添加如下步骤。
![blob.png](http://file.ctosb.com/upload/image/20170429/1493453783559040557.png "1493453783559040557.png")

![blob.png](http://file.ctosb.com/upload/image/20170429/1493453815023061168.png "1493453815023061168.png")

这样，Jenkins就将代码规范、findbug、代码覆盖率、单元测试结果收集起来了，并且可以在Jenkins中以图表的方式展示。如下图

![blob.png](http://file.ctosb.com/upload/image/20170429/1493454092111079282.png "1493454092111079282.png")

4. 将检查后结果发送邮件给对应开发人员

在《构建后操作》中增加发送邮件步骤。如下图
![blob.png](http://file.ctosb.com/upload/image/20170429/1493454264891093441.png "1493454264891093441.png")

这样会将构建日志和fingbug结果信息发送到对应的开发人员。细心点的人员可能会发现，我在这里填的收件人、主题、内容似乎都是变量，那实际的值都是什么呢？特别是收件人，怎么知道提交代码人的邮箱？

由于Gitlab的帐号默认就是邮箱帐号，并且邮箱字段是必填的，所以Jenkins会自动将提交代码人员的邮箱帐号作为邮箱。以下是Jenkins的系统设置截图
![blob.png](http://file.ctosb.com/upload/image/20170429/1493455284543029875.png "1493455284543029875.png")

由于Gitlab的邮箱帐号是通过AD域自动生成的，但该邮箱并不是公司的邮箱，所以在发送邮件之前统一替换收件人邮箱帐号，使用的脚本语法是groovy。

基本上这样，就完成了一个自动化构建、代码规范检测、findbug查找、单元测试。即开发人员每次提交代码后会收到一个构建结果邮件,如下图。

![blob.png](http://file.ctosb.com/upload/image/20170429/1493456045247065558.png "1493456045247065558.png")
