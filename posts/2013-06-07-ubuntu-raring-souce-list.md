---
layout: post
title: '让ubuntu使用国内的源'
description: '让ubuntu使用国内的源，加快更新速度'
date: 2013-06-07
comments: true
categories:
- Linux
tags:
- ubuntu 
- raring
- source list

---
如果你安装了ubuntu 13.04，但是没有选择中文语言，你默认的源将会是us的。这样速度不太理想，如果你想使用国内源，请替换/etc/apt/source.list文件为下面的内容：


```
deb http://cn.archive.ubuntu.com/ubuntu/ raring main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ raring-security main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ raring-updates main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ raring-proposed main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ raring-backports main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ raring main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ raring-security main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ raring-updates main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ raring-proposed main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ raring-backports main restricted universe multiverse
```