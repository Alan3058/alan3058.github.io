---
layout: post
title:  "redis install"
categories: [redis]
tags: [redis]
fullview: false
published: false
---

# Prepare
Machine number: 3 (ip:192.168.0.10,192.168.0.11,192.168.0.12)   
Machine exposed port: 6379  
Operating System: Centos 7    
Redis: 3.2.11  

# Redis install
## Install it
Run following command,to download and compile and install redis.  
```shell
wget http://download.redis.io/releases/redis-3.2.11.tar.gz
tar -zxvf redis-3.2.11.tar.gz 
cd redis-3.2.11 && make
```
and then, you will look the success install infomation on the screen.


## Run it
Run the following command on the machine(192.168.0.10).
> $ ./src/redis-server

and then, you will look the success start infomation on the screen.

## Check it
Now, you can run the following command to check whether the redis server startup is successful.  
connect the redis server
> $ ./src/redis-cli -h 127.0.0.1 -p 6379

lookup the redis keys
> $ keys *



# Redis master slave config

## Master slave config
* The first way  
Run the command on the slave machines.
> $ ./src/redis-server --port 6379 --slaveof <server-ip> <port>

eg: 
run it on the slave192.168.0.11. 
> $ ./src/redis-server --port 6379 --slaveof 192.168.0.10 6379

run it on the slave192.168.0.12
> $ ./src/redis-server --port 6379 --slaveof 192.168.0.10 6379


* The second way  
Or edit slave config file
```
slaveof <server-ip> <port>
```
And then run
> $ ./src/redis-server 

eg: edit redis.conf of the slave machines(192.168.0.11,192.168.0.12)
```
slaveof 192.168.0.10 6379
```

run it on the slave machines
> $ ./src/redis-server --port 6379
## Check it
Run add data command on the master machine, and then lookup whether the data exists from the slave machine.
> $ set test 1

check the data on the slave machine
> $ get test

