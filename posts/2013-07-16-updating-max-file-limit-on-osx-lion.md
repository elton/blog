---
layout: post
title: '更新OSX中的max file的limit'
description: '加大Mac OSX中的max file值'
date: 2013-07-16
comments: true
categories:
- Mac
tags:
- max file 
- limit
- macos

---
默认的情况下，OSX的最大文件数只有256：


```
$ launchctl limit
```

```
	cpu         unlimited      unlimited      
    filesize    unlimited      unlimited      
    data        unlimited      unlimited      
    stack       8388608        67104768       
    core        0              unlimited      
    rss         unlimited      unlimited      
    memlock     unlimited      unlimited      
    maxproc     709            1064           
    maxfiles    256            unlimited
```

编辑/etc/launchctl.conf文件，使设置永久有效

```
$ sudo vim /etc/launchctl.conf
```

在文件的末尾加上

```
limit maxfiles 16384 32768
```

重启系统，设置就生效了。