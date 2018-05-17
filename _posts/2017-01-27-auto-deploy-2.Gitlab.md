---
layout: post
title: [自动化发布-2.Gitlab环境搭建]
categories: [自动化发布]
tags: [自动化发布,gitlab,环境搭建]
id: [18872430231552]
fullview: false
---
做完portal门户网站，老大开始布置下一个任务，建立Gitlab代码仓库来替换svn仓库，未来所有项目代码都将托管于Gitlab仓库。想了解Gitlab，大家可网上自行百度搜索下，Gitlab其实就是一个开源的Github。

### 安装Docker

安装Gitlab过程大致有两种方式，一种离线rpm，一种是通过docker容器方式，这里我选择了docker方式安装。按照官方文档，首先我们需要安装docker，安装docker步骤详见[自动化发布-docker安装](/170212/auto-deploy-8.Docker-install)。

### 安装Gitlab

安装完docker后，接着安装运行Gitlab镜像。执行以下命令，docker引擎将会自动下载安装Gitlab镜像。
```bash
$ sudo docker run --detach \
--hostname gitlab.example.com \
--publish 443:443 --publish 80:80 --publish 22:22 \
--name gitlab \
--restart always \
--volume /srv/gitlab/config:/etc/gitlab \
--volume /srv/gitlab/logs:/var/log/gitlab \
--volume /srv/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```
### Gitlab数据文件描述

gitlab数据存储映射如下  

| 主机位置 | 容器位置 | 说明 |
| - | - | - | 
| /srv/gitlab/data | /var/opt/gitlab | 存储Gitlab数据 |
| /srv/gitlab/logs | /var/log/gitlab | 存储Gitlab日志 |
| /srv/gitlab/config | /etc/gitlab | 存储Gitlab配置 |

### Gitlab配置文件

由于是这个容器是使用官方的Omnibus GitLab安装包，因此所有的配置都是在/etc/gitlab/gitlab.rb文件里面做。

编辑配置文件有两种方式。

第一种：直接进入docker容器，找到配置文件编辑即可
> $ docker exec -it gitlab /bin/bash

第二种：直接编辑配置文件

> $ docker exec -it gitlab vi /etc/gitlab/gitlab.rb

注意：每次修改Gitlab配置文件后，都需要执行如下命令，以便配置能够生效。

> $ gitlab-ctl reconfigure $ gitlab-ctl restart

### 配置smtp

修改配置文件/etc/gitlab/gitlab.rb对应内容如下
```yaml
gitlab_rails['gitlab_email_from'] = 'git@ctosb.com' 
gitlab_rails['smtp_enable'] = true 
gitlab_rails['smtp_address'] = "smtp.ctosb.com" 
gitlab_rails['smtp_port'] = 25 
gitlab_rails['smtp_user_name'] = "git@ctosb.com" 
gitlab_rails['smtp_password'] = "password" 
gitlab_rails['smtp_domain'] = "smtp.ctosb.com" 
gitlab_rails['smtp_authentication'] = "login" 
gitlab_rails['smtp_enable_starttls_auto'] = true
```

### 配置ldap

修改配置文件为/etc/gitlab/gitlab.rb如下
```yaml
gitlab_rails['ldap_enabled'] = true 
gitlab_rails['ldap_servers'] = YAML.load <<-'EOS' /# remember to close this block with 'EOS' below 
 main: /# 'main' is the GitLab 'provider ID' of this LDAP server 
  label: 'LDAP' 
  host: '192.168.16.226' 
  port: 389 
  uid: 'sAMAccountName' 
  method: 'plain' /# "tls" or "ssl" or "plain" 
  bind_dn: 'liliangang@ctosb.com' 
  password: 'abcd1234' 
  active_directory: true 
  allow_username_or_email_login: true 
  block_auto_created_users: false 
  base: 'ou=Accounts,dc=ctosb,dc=com' 
  user_filter: '' 
EOS
```

参考文档：[Gitlab官方安装文档](https://docs.gitlab.com/omnibus/docker/)
