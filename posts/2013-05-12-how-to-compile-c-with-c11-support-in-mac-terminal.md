---
layout: post
title: ' Mac Terminal如何支持C++11'
description: ''
date: 2013-05-12
comments: true
categories:
- C
tags:
- C/C++ 
- compiler

---

如果是用g++编译C++11文件，会出现下面问题

```
$ g++ -std=c++11 string.cc -o string
cc1plus: error: unrecognized command line option "-std=c++11"

$ g++ -v

…

gcc version 4.2.1 (Based on Apple Inc. build 5658) (LLVM build 2336.11.00)
```

原因是Mac自带的g++版本太低。 如果想使用C++11，可以用clang++替代g++，并用libc++替换libstdc++，因为libstdc++的版本也太老，不支持c++11

```
clang++ -std=c++11 -stdlib=libc++ -Weverything main.cpp
```
这样就可以正常编译C++11的文件了

