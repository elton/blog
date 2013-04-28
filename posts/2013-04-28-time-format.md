---
layout: post
title: 'golang time.Time.Format 使用说明'
description: 'golang, time'
date: 2013-04-28
comments: true
categories:
- Go
tags:
- Go
- time
- format

---
今天用到golang的time包了,使用到了time.Time对象,但是Time的Format方法搞了半天也没用明白怎么用,去网上找也没到,郁闷之极.

根据doc看到time.RFC3339，输出的内容为：

```
2006-01-02T15:04:05Z07:00
```

所以联想到，go中的时间表示方法应该是：

* 月 - 1
* 日 - 2
* 时 - 3（如果是24小时制，就是15）
* 分 - 4
* 秒 - 5
* 年 - 6
* 时区 - 7

感觉很神奇啊。

