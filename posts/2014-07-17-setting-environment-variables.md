---
layout: post
title: 'Mac OSX下设置IntelliJ IDEA环境变量'
description: '如果设置环境变量，让GUI可用'
date: 2014-07-17
comments: true
categories:
- Mac
tags:
- Environment 
- Variable

---

配置InelliJ时候，发现自己在~/.profile中设置的环境变量都不好用，比如**M2_HOME**, 后来找到了一个方法，只有这么设置，才能在GUI环境下使用环境变量，之前profile中的，只有在命令行中才有用

```
launchctl setenv MYPATH myvar
```