---
layout: post
title: '给Mac OSX的Dock上加入最近打开的应用功能'
description: ''
date: 2013-05-30
comments: true
categories:
- Mac
tags:
- Mac 
- Dock

---

如果你想把你的Dock加入一个类似于最近打开的应用的功能，方便快速打开最近使用的应用的话，有什么办法吗？

其实很简单，只要在terminal中输入以下命令就好了

```
defaults write com.apple.dock persistent-others -array-add '{ "tile-data" = { "list-type" = 1; }; "tile-type" = "recents-tile"; }'
```

然后在重启Dock

```
killall Dock
```

![recent apps](http://cdn.cultofmac.com/wp-content/uploads/2013/05/Dock-Stacks-Recent-Apps.jpg)