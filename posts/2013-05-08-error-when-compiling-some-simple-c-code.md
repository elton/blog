---
layout: post
title: 'C++代码编译时出现 ld: symbol(s) not found for architecture x86_64错误'
description: ''
date: 2013-05-08
comments: true
categories:
- C
tags:
- C/C++ 
- compiler

---

当编译c++代码时候，出现

```
 ld: symbol(s) not found for architecture x86_64
```

上面错误时，一般是因为使用C的front-end去编译C++代码。使用gcc编译C++代码，它没有链接C++的liberies.例如：

```
$ gcc example.cpp 
Undefined symbols for architecture x86_64:
  "std::cout", referenced from:
      _main in ccLTUBHJ.o
  "std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)", referenced from:
      _main in ccLTUBHJ.o
  "std::basic_ostream<char, std::char_traits<char> >& std::endl<char, std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&)", referenced from:
      _main in ccLTUBHJ.o
  "std::basic_ostream<char, std::char_traits<char> >::operator<<(std::basic_ostream<char, std::char_traits<char> >& (*)(std::basic_ostream<char, std::char_traits<char> >&))", referenced from:
      _main in ccLTUBHJ.o
  "std::ios_base::Init::Init()", referenced from:
      __static_initialization_and_destruction_0(int, int)in ccLTUBHJ.o
  "std::ios_base::Init::~Init()", referenced from:
      ___tcf_0 in ccLTUBHJ.o
ld: symbol(s) not found for architecture x86_64
collect2: ld returned 1 exit status
$ g++ example.cpp 
$ 
```
使用g++就不会出现这个问题了。

使用clang也会出现类似的问题。

```
$ clang example.cpp 
Undefined symbols for architecture x86_64:
  "std::ios_base::Init::~Init()", referenced from:
      ___cxx_global_var_init in cc-IeV9O1.o
  "std::ios_base::Init::Init()", referenced from:
      ___cxx_global_var_init in cc-IeV9O1.o
  "std::cout", referenced from:
      _main in cc-IeV9O1.o
  "std::basic_ostream<char, std::char_traits<char> >& std::endl<char, std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&)", referenced from:
      _main in cc-IeV9O1.o
  "std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)", referenced from:
      _main in cc-IeV9O1.o
  "std::ostream::operator<<(std::ostream& (*)(std::ostream&))", referenced from:
      _main in cc-IeV9O1.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
$ clang++ example.cpp 
$
```

就如上面提示，加入-v参数，会提示类似下面内容

```
"/usr/bin/ld" -demangle -dynamic -arch x86_64 
    -macosx_version_min 10.6.8 -o a.out -lcrt1.10.6.o
    /var/folders/zl/zlZcj24WHvenScwjPFFFQE+++TI/-Tmp-/cc-hdOL8Z.o
    -lSystem /Developer/usr/bin/../lib/clang/3.0/lib/darwin/libclang_rt.osx.a
```
如果替换成clang++后，提示就变为

```
"/usr/bin/ld" -demangle -dynamic -arch x86_64
     -macosx_version_min 10.6.8 -o a.out -lcrt1.10.6.o 
     /var/folders/zl/zlZcj24WHvenScwjPFFFQE+++TI/-Tmp-/cc-wJwxjP.o 
     /usr/lib/libstdc++.6.dylib -lSystem
     /Developer/usr/bin/../lib/clang/3.0/lib/darwin/libclang_rt.osx.a
```