---
layout: post
title:  "redis install"
categories: [redis]
tags: [redis]
fullview: false
published: true
---

# 1. Prepare
Machine number: 3 (ip:192.168.0.10,192.168.0.11,192.168.0.12)   
Machine exposed port: 6379  
Operating System: Centos 7    
Redis: 3.2.11  

# 2. Redis install
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
```shell
$ ./src/redis-server
```

and then, you will look the success start infomation on the screen.

## Check it
Now, you can run the following command to check whether the redis server startup is successful.  
connect the redis server
```shell
$ ./src/redis-cli -h 127.0.0.1 -p 6379
```

lookup the redis keys
```shell
$ keys *
```


# 3. Redis master slave
## Update the master config file.  
Create or update the following infomation.
```
bind 0.0.0.0  
or  edit as follow
bind [master-ip]
```

## Update the slave config file.  
1. The first way  
Run the command on the slave machines.  
```shell
$ ./src/redis-server --port 6379 --slaveof [server-ip] [port]
```
eg:  
run it on the slave192.168.0.11.  
```shell
$ ./src/redis-server --port 6379 --slaveof 192.168.0.10 6379
```
run it on the slave192.168.0.12   
```shell
$ ./src/redis-server --port 6379 --slaveof 192.168.0.10 6379
```
2. The second way  
Or edit slave config file
```
slaveof <server-ip> <port>
```
And then run  
```shell
$ ./src/redis-server 
```
eg: edit redis.conf of the slave machines(192.168.0.11,192.168.0.12)  
```
slaveof 192.168.0.10 6379
```
run it on the slave machines  
```shell
$ ./src/redis-server --port 6379
```

## Check it
Run add data command on the master machine, and then lookup whether the data exists from the slave machine.
```shell
$ set test 1
```
check the data on the slave machine  
```shell
$ get test
```
if the slave is shutdown, then have not effect; else the master is shutdown, then will not available.  

# 4. Redis cluster
## Prepare
cluster config directory: /app/redis-3.2.11/cluster-conf/[port]  
cluster data directory: /app/redis-data/[port]  
port:7001,7002  
 
## Create directory
create config directory
```shell
mkdir -p /app/redis-3.2.11/cluster-conf/7001
mkdir -p /app/redis-3.2.11/cluster-conf/7002
```
create data directory
```shell
mkdir -p /app/redis-data/7001
mkdir -p /app/redis-data/7002
```

## Create config file
copy the config file to the cluster config directory.  
```
cp /app/redis-3.2.11/redis.conf /app/redis-3.2.11/cluster-conf/7001
cp /app/redis-3.2.11/redis.conf /app/redis-3.2.11/cluster-conf/7002
```
edit the config file of 7001,as following  
```
port 7001
logfile "/app/redis-data/7001/redis.log"
dir /app/redis-data/7001/
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
bind 0.0.0.0
```
edit the config file of 7002,as following   
```
port 7002
logfile "/app/redis-data/7002/redis.log"
dir /app/redis-data/7002/
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
bind 0.0.0.0
```

## Copy the redis to the other server
```shell
scp -r redis-3.2.11/ root@192.168.0.11:/app/
scp -r redis-3.2.11/ root@192.168.0.12:/app/
```

## Run all of redis server
Run the following command on every server.
```shell
./src/redis-server /app/redis-3.2.11/cluster-conf/7001/redis.conf &
./src/redis-server /app/redis-3.2.11/cluster-conf/7002/redis.conf &
```

## Creating cluster
Run the following command on a redis server.  
```shell
./redis-trib.rb create --replicas 1 192.168.0.10:7001 192.168.0.10:7001 192.168.0.11:7001 192.168.0.11:7002 192.168.0.12:7002 192.168.0.12:7002
```
Tip: the redis server must install ruby , redis dependcy package of ruby.  
```shell
yum install -y ruby rubygems
# network error，gem install local redis lib
gem install --local redis-3.3.5.gem
```

There are some fault in install ruby, we also can use rvm style to install ruby.  You can click the [ruby version manager](http://rvm.io/rvm/install)


## Check it
Run the following command to connect cluster redis.  
```shell
$ ./src/redis-cli -c -p 7001
```
Run the following command to show the cluster infomation.  
```shell
cluster nodes
```
For example, the following infomation  
```shell
1104bda669874620bfb31d69ec166d99c086fd4b 192.168.0.10:7002 slave 93e71137f7ff724f587bfa1c1054d721617c9b7c 0 1527061405087 2 connected
93e71137f7ff724f587bfa1c1054d721617c9b7c 192.168.0.11:7001 myself,master - 0 0 2 connected 5461-10922
c0c4bfe56bf2627706cc5a4420b00b43288cedfd 192.168.0.10:7001 master - 0 1527061405592 1 connected 0-5460
4cf2f4082e7fd517973ad162af44f9c6b331f7f6 192.168.0.12:7001 master - 0 1527061404079 3 connected 10923-16383
b4164c29660a2990fe1876a4afb6bb26bb50deab 192.168.0.12:7002 slave 4cf2f4082e7fd517973ad162af44f9c6b331f7f6 0 1527061404584 6 connected
7c9873e20ab729ce69c2839d7aa84dd7cd0f183c 192.168.0.11:7002 slave c0c4bfe56bf2627706cc5a4420b00b43288cedfd 0 1527061404079 5 connected
```
And Now, you are successful to create the cluster redis server.

# 5. Simple Redis cluster
1. Enter create cluster directory
```shell
cd redis-3.2.11/utils/create-cluster
```

2. Start redis instance
```shell
./create-cluster start && ./create-cluster create
```

3. Stop and clean redis cluster
```shell
./create-cluster stop && ./create-cluster clean
```
