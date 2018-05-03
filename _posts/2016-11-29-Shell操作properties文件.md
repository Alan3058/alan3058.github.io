---
layout: post
title: [Shell操作properties文件]
categories: [Linux学习]
tags: [shell,操作properties：properties]
id: [18855653015552]
fullview: false
---
Shell读取properties文件属性值

例如读取config.properties文件的jdbc.url属性值

```bash
$    awk -F= '/jdbc.url/{print $2}' config.properties
```

替换修改properties文件属性值

例如修改config.properties文件的jdbc.url属性值为testurl

```bash
$    sed -i "s#^jdbc.url=.*#jdbc.url=testurl#g"  config.properties
```


