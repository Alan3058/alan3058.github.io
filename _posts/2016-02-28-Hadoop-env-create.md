---
layout: post
title: [Hadoop环境搭建]
categories: [bigdata]
tags: [hadoop,java,搭建]
id: [18654318034944]
fullview: false
---

# 背景

大数据，这个词对于我们IT人员来说，应该是天天在耳边吹的一个词语，就目前和未来的发展形势来看，绝对是势不可挡。现如今关于BAT三巨头的新闻，也经常会提到大数据分析、挖掘等高端词汇![](http://img.baidu.com/hi/face/i_f07.gif)![](http://img.baidu.com/hi/face/i_f07.gif)![](http://img.baidu.com/hi/face/i_f07.gif)，与大数据相关的词汇Hadoop、Storm、Spark、NoSql、MongoDb、Redis、GFS、MapReduce。。。太多了相关技术词语，对于渣渣技术人员来说，感觉真心学不过来![](http://img.baidu.com/hi/face/i_f08.gif)![](http://img.baidu.com/hi/face/i_f08.gif)![](http://img.baidu.com/hi/face/i_f08.gif)。。。然而为了不被跟上时代步伐，不脱大众后腿，只能硬着头皮去了解下![](http://img.baidu.com/hi/face/i_f32.gif)![](http://img.baidu.com/hi/face/i_f32.gif)![](http://img.baidu.com/hi/face/i_f32.gif)。经过一番纠结对比，最后决定先从Hadoop开刀。。。废话到此为止![](http://img.baidu.com/hi/face/i_f30.gif)![](http://img.baidu.com/hi/face/i_f30.gif)![](http://img.baidu.com/hi/face/i_f30.gif)

# 环境


操作系统：本地Windows7 64位


 Linux Centos6.5 64位

软件：Hadoop 2.7.2

 Java 1.8.0_74 64位

似乎Hadoop每个版本的配置变化都很大，故作者选择当前最新版Hadoop2.7.2。然而似乎噩梦来了Apache官网一般对于最新版本的软件没有中文版文档，最后只能硬着头皮啃英文文档了![](http://img.baidu.com/hi/face/i_f09.gif)![](http://img.baidu.com/hi/face/i_f09.gif)![](http://img.baidu.com/hi/face/i_f09.gif)

# 开始

首先Hadoop分三种模式：单机模式（不集群）、单机伪分布式模式（同一台主机上部署多个应用集群）、分布式模式（多台机器集群，实际生成应用环境）。本文从单机模式开始。


1.首先需要安装Java,这个步骤可以从网上找到一大堆。

2.安装ssh服务，并配置ssh无密码登录(单机模式下可不配置，伪分布式和分布式需要配置)。

安装ssh服务大家可以直接用yum安装，很简单，这里略过，下面介绍配置ssh无密码登录。

创建ssh_key,这里采用dsa方式。

```bash
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
```

（注:执行完该命令，会在~/.ssh/下产生两个文件id_dsa和id_dsa.pub）

将id_dsa.pub追加到authorized_keys文件中

```bash
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
```

修改authorized_keys文件权限（可选步骤）

```bash
$ chmod 0600 ~/.ssh/authorized_keys
```

这是可以直接无密码连接登录


```bash
$ ssh localhsot
```

3.安装Hadoop

官网下载Hadoop软件包，解压即可。首先Hadoop有两大重要组件：一是Hdfs（对应谷歌的GFS），另外一个是MapReduce（并行计算框架）。

修改hadoop环境配置文件etc/hadoop/hadoop-env.sh文件的java_home路径。

```java
export JAVA_HOME=/usr/java/jdk1.8.0_74
```

输入一下命令将会显示hadoop命令使用的方法

```bash
$ bin/hadoop
```

## 单机模式

默认情况下，Hadoop就是支持单机模，并且可以断点Debug调试。如下命令，将会统计文件夹下的所有文件中某个字符的个数。


首先将构建input文件夹，并拷贝文件到该文件夹中

```bash
$ mkdir input
$ cp etc/hadoop/*.xml input
```

运行hadoop运算,计算所有dfs英文单词的个数，并将结果输出到output文件夹中

```bash
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z]+'
```

查看计算结果

```bash
cat output/*
```

## 单机伪分布式模式


首先配置ssh无密钥登录，见上

### 1.HDFS文件系统构建

配置etc/hadoop/core-site.xml文件

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

配置etc/hadoop/hdfs-site.xml


```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

格式化hdfs文件系统


```bash
$ bin/hdfs namenode -format
```

启动hdfs服务

```bash
$ sbin/start-dfs.sh
```

输入url,验证是否启动成功(注：如果是其他机器访问该url，请确保端口已开放)

```java
http://localhost:50070
```

在hdfs文件系统中创建用户路径


```bash
$ bin/hdfs dfs -mkdir /user
$ bin/hdfs dfs -mkdir /user/alan
```

拷贝本地input下文件到hdfs文件系统中

```bash
$ bin/hdfs dfs -put etc/hadoop input
```

运行计算单词的案例

```bash
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
```

输出计算结果


```bash
$ bin/hdfs dfs -cat output/*
```

或者将输出结果写出到本地，然后查看

```bash
$ bin/hdfs dfs -get output output
$ cat output/*
```

### YARN安装

配置etc/hadoop/mapred-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

配置etc/hadoop/yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

启动yarn服务

```bash
$ sbin/start-yarn.sh
```

输入url，校验yarn是否启动成功(注：如果是其他机器访问该url，请确保端口已开放)


```java
http://localhost:8088
```

至此，Hadoop单机模式和单机伪分布模式的环境已搭建成功。


