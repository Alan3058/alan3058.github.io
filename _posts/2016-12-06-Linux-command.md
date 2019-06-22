---
layout: post
title: [Linux常用命令]
categories: [Linux]
tags: [linux,常用命令]
id: [18859847319552]
fullview: false
---

ssh端口代理设置

```bash
$  ssh -N -f -L 6666:10.163.12.13:1521 gelcmw@10.163.67.31
```

查看文件下某个字符，并显示前后50行。

```bash
$  grep  -50 'testchar'   text.txt
```

查看目录下文件及文件夹大小，并排序

```bash
$  du -sk * | sort -n
```


