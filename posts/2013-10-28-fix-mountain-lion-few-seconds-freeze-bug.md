---
layout: post
title: '修复 Mountain Lion 短暂假死问题'
description: '修复 Mountain Lion 短暂假死问题'
date: 2013-10-28
comments: true
categories:
- Mac
tags: 
- macos

---


在 Mountain Lion 下正常使用时，有时会出现几秒种鼠标不能动、按键没反应的假死状态。今天恰好看到 [Hacker News 上一篇博文提到这个问题，并给出了解决方案][1]。 

   [1]: https://news.ycombinator.com/item?id=5625977

![][2]

   [2]: http://cdn.lucifr.com/uploads/ML_mdworker_error.png (mdworker errors)

如果你也遇到类似问题，并且在 Console 里能看到大量 “sandboxd: mdworker deny mach-lookup com.apple.ls.boxd”、“mdworker: Unable to talk to lsboxd” 这样的错误信息，那么可以试试这个解决方案： 

  1. 重新启动 OS X，按住 `Shift` 键进入[安全模式][3]。注意需要启动就按住 Shift 键不要松开，成功进入安全模式的标志是看到苹果标志下有显示进度条。
  2. 登入后再重新启动，这次常规启动。
  3. 登入后查看下 Console，成功的话那些错误信息应该不会出现了。

   [3]: http://support.apple.com/kb/HT1564?viewlocale=zh_CN (Mac OS X：“安全启动”和“安全模式”是什么？)

此外[原文中][4]还给出了另外一种方法，**不过需要修改系统文件，存在一定风险因此 Lucifr 并不推荐。如果安全模式的方法不起作用，那么也可以一试，一定注意事先做好备份，搞坏了 Lucifr 不负责**。具体如下： 

   [4]: http://www.princeton.edu/~jcjb/docs/osx_error_fix/ (how to fix the sandboxd mdworker deny mach-lookup com.apple.ls.boxd error)

修改 `/System/Library/Sandbox/Profiles` 目录下的 system.sb 文件，加入以下内容： 
    
    (allow mach-lookup (global-name "com.apple.ls.boxd"))
    
    (allow mach-lookup (local-name "com.apple.ls.boxd"))
    

此外作者也推测了发生这个问题的原因，认为可能是因为升级的缘故，造成 Mountain Lion 的沙盒机制将 Spotlight 排除在信任名单之外了，而在进入安全模式后，一些缓存文件得到了清除和重建，从而解决这个问题。 
