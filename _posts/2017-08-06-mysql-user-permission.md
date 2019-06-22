---
layout: post
title: [mysql数据库用户权限管理]
categories: [DB]
tags: [mysql,DB用户权限管理]
id: [19967786419748864]
fullview: false
---

### 常用操作

查询mysql用户信息

```bash
select * from mysql.user
```

新增用户

```bash
create user username IDENTIFIED by 'password'
```

修改用户名称

```bash
rename  user  old-username to new-username
```

修改用户密码

```bash
update user set `Password` = PASSWORD('test') where `User` = 'username'
```

删除用户

```bash
drop user username
```

查看用户权限

```bash
show grants for username
```

授权用户部分权限


```bash
grant SELECT,UPDATE,DELETE,INSERT on test.* to username
```

回收用户部分权限


```bash
revoke UPDATE,DELETE,INSERT on test.* to username
```

以上操作如果未生效，可执行如下语句刷新缓存

```bash
flush  privileges
```

### user表host字段含义

host字段表示连接到mysql服务器的连接ip信息。

% 匹配所有主机

localhost localhost不会被解析成IP地址，直接通过UNIXsocket连接

127.0.0.1 会通过TCP/IP协议连接，并且只能在本机访问；

::1 ::1就是兼容支持ipv6的，表示同ipv4的127.0.0.1

限制用户只能本地连接。

```bash
update user set `host` = 'localhost' where `User` = 'username'
```

限制某个区段的ip可以访问

```bash
update user set `host` = '192.168.12.%' where `User` = 'username'
```

参考：[http://www.cnblogs.com/fslnet/p/3143344.html](http://www.cnblogs.com/fslnet/p/3143344.html) 

