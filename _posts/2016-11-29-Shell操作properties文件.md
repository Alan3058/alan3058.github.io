---
layout: post
title: [Shell操作properties文件]
categories: [Linux学习]
tags: [shell,操作properties：properties]
fullview: false
---
Shell读取properties文件属性值

例如读取config.properties文件的jdbc.url属性值
$ awk -F= '/jdbc.url/{print $2}' config.properties

替换修改properties文件属性值

例如修改config.properties文件的jdbc.url属性值为testurl
$ sed -i "s/#^jdbc.url=./*/#jdbc.url=testurl/#g" config.properties
