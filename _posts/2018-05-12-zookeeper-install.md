---
layout: post
title:  "zookeeper install"
categories: [Middleware,zookeeper]
tags: [java,zookeeper]
fullview: false
published: true
---

The zookeeper is a stronger midware, and it a high-perfomance coordination service for distributed applications.It exposed common service - such as naming, configuration management, synchronization,and group services. You can create the distribute lock, distribute configuration file manage, and the other distribute thing by using it.  
I will to install the zookeeper next.

# 1. Prepare
**Machine number:** 3 (ip:192.168.0.10,192.168.0.11,192.168.0.12)   
**Machine exposed port:** 2181(client port), 2788(used by machine communication), 3788(used by machine communication)    
**Operating System:** Centos 7    
**Java:** openjdk1.8    
**zookeeper:** 3.4.12    

> tip: To ensure the communication of the 3 machine and the client can connection zookeeper service of the machine. we must close the firewall on the machine or open the port(above:2181,2788,3788) on the machine.


# 2. Standalone
## download zookeeper
Click the [zookeeper](http://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz) url to download.

## unzip file  
Unzip file to your app path.(eg:/app/zookeeper)

## create config file  
Go into the `conf` directory in the zookeeper path ,copy the `zoo_sample.cfg` file to **`zoo.cfg`** file.  
For the linux ,you can input the follow:
> cp zoo_sample.cfg zoo.cfg

In the file, There is a default client port that value of 2181, you can connection the zookeeper by the port.

## start the zookeeper  
Go into the `bin` directory in the zookeeper path, and input the follow:  
> sh zkServer.sh start

Or  
> sh zkServer.sh start-foreground

Now, you can connection all of machine.input the follow command. (eg: the ip is 192.168.0.10, the port is 2181)  
> sh zkCli.sh -server 192.168.0.10:2181  

tip: you also add the `bin` directory in the zookeeper path to the environment variable. looke the follow:  
> export PATH="$PATH:/app/zookeeper/bin"

# 3. Master slave 
## prepare
The First, you must have more the three machine. and the odd number of machine is the  best practice, such as three, five, seven.(eg: I use the three machine)

## define the unique id by every machine  
Define the unique id by every machine.(eg: 1->192.168.0.10, 2->192.168.0.11, 3->192.168.0.12)  

## update the `config` file by every machine  
update content below the follow:  
```
dataDir=/app/zk/data # data directory
dataLogDir=/app/zk/log # log directory

# the machine define 
server.1=192.168.0.10:2788:3788
server.2=192.168.0.11:2788:3788
server.3=192.168.0.12:2788:3788
```

## create `myid` file  
Create a `myid` file in the data directory of zookeeper, and the file must contain the value of the machine id  by predefined.    
Such as the follow:  
The first machine's id value is 1.
> echo 1 > myid  

The second machine's id value is 2.  
> echo 2 > myid  

The third machine's id value is 3.  
> echo 3 > myid  

## Start all of machine  
Input the command by every machine:  
> sh zkServer.sh start  

Or  
> sh zkServer.sh start-foreground  

Now, you can connection all of machine by the follow command.  
> sh zkCli.sh -server 192.168.0.10:2181,192.168.0.11:2181,192.168.0.12:2181

also lookup every machine's mode by the follow command
> sh zkServer.sh status
