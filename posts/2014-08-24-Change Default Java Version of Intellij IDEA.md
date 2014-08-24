---
layout: post
title: 'Mac系统修改Intellij Idea默认JDK版本'
description: 'Intellij IDEA 默认情况下，使用的jdk的版本是1.6，当第一次启动IDEA的时候，如果系统中未安装jdk，则系统会自动到苹果官网下载jdk安装文件。如果你的系统已经安装了jdk1.7或是更高的版本，同样首次打开IDEA的时候要求你安装苹果官网jdk1.6。'
date: 2014-08-24
comments: true
categories:
- Java
tags:
- Idea 
- JDK
- Version

---

Intellij IDEA 默认情况下，使用的jdk的版本是1.6，当第一次启动IDEA的时候，如果系统中未安装jdk，则系统会自动到苹果官网下载jdk安装文件。如果你的系统已经安装了jdk1.7或是更高的版本，同样首次打开IDEA的时候要求你安装苹果官网jdk1.6。

为了免去多余的jdk安装，解决办法如下：

 

到/Applications下找到IntelliJ IDEA 13，右键－>显示包内容－>Contents->Info.plist，利用文本编辑器或是默认的xcode打开该文件，找到下列代码

```
<key>JVMVersion</key>
<string>1.6*</string>

将<string>1.6*</string>改为<string>1.7*</string>保存。
```
 

此时idea使用的jdk就是1.7及以上的版本了。