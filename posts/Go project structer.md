---
layout: post
title: 'Go项目的目录结构'
description: 'Go'
date: 2013-04-20
comments: true
categories:
- Go
tags:
- Go

---

{:toc}

项目目录结构如何组织，一般语言都是没有规定。但Go语言这方面做了规定，这样可以保持一致性

# 一般的，一个Go项目在GOPATH下，会有如下三个目录：
```
|--bin
|--pkg
|--src
```
其中，bin存放编译后的可执行文件；pkg存放编译后的包文件；src存放项目源文件。一般，bin和pkg目录可以不创建，go命令会自动创建（如 go install），只需要创建src目录即可。
对于pkg目录，曾经有人问：我把Go中的包放入pkg下面，怎么不行啊？他直接把Go包的源文件放入了pkg中。这显然是不对的。pkg中的文件是Go编译生成的，而不是手动放进去的。（一般文件后缀.a）
对于src目录，存放源文件，Go中源文件以包（package）的形式组织。通常，新建一个包就在src目录中新建一个文件夹。

# 举例说明
比如：我新建一个项目，test，开始的目录结构如下：

```
test/
|--src
```
为了编译方便，我在其中增加了一个install文件，目录结构：

```
test/
|-- install
`-- src
```
其中install的内容如下：（linux下）

```
#!/usr/bin/env bash

if [ ! -f install ]; then
echo 'install must be run within its container folder' 1>&2
exit 1
fi

CURDIR=`pwd`
OLDGOPATH="$GOPATH"
export GOPATH="$CURDIR"

gofmt -w src

go install test

export GOPATH="$OLDGOPATH"

echo 'finished'
```
之所以加上这个install，是不用配置GOPATH（避免新增一个GO项目就要往GOPATH中增加一个路径）

接下来，增加一个包：config和一个main程序。目录结构如下：

```
test
|-- install
`-- src
    |-- config
    |   `-- config.go
    `-- test
        `-- main.go
```
注意，config.go中的package名称必须最好和目录config一致，而文件名可以随便。main.go表示main包，文件名建议为main.go。（注：不一致时，生成的.a文件名和目录名一致，这样，在import 时，应该是目录名，而引用包时，需要包名。例如：目录为myconfig，包名为config，则生产的静态包文件是：myconfig.a，引用该包：import “myconfig”，使用包中成员：config.LoadConfig()）

config.go和main.go的代码如下：
config.go代码

```
package config

func LoadConfig() {

}
```

main.go代码

```
package main

import (
    "config"
    "fmt"
)

func main() {
    config.LoadConfig()
    fmt.Println("Hello, GO!")
}
```
接下来，在项目根目录执行./install

这时候的目录结构为：

```
test
|-- bin
|   `-- test
|-- install
|-- pkg
|   `-- linux_amd64
|       `-- config.a
`-- src
    |-- config
    |   `-- config.go
    `-- test
        `-- main.go
```
（linux_amd64表示我使用的操作系统和架构，你的可能不一样）
其中config.a是包config编译后生成的；bin/test是生成的二进制文件

这个时候可以执行：bin/test了。会输出：Hello, GO!

3、补充说明
1）包可以多层目录，比如：net/http包，表示源文件在src/net/http目录下面，不过源文件中的包名是最后一个目录的名字，如http
而在import包时，必须完整的路径，如：import “net/http”

2）有时候会见到local import（不建议使用），语法类似这样：

```
import “./config”
```
当代码中有这样的语句时，很多时候都会见到类似这样的错误：

```
local import “./config” in non-local package
```
我所了解的这种导入方式的使用是：当写一个简单的测试脚本，想要使用go run命令时，可以使用这种导入方式。

比如上面的例子，把test/main.go移到src目录中，test目录删除，修改main.go中的import “config”为import “./config”，然后可以在src目录下执行：go run main.go

可见，local import不依赖于GOPATH

摘自：http://blog.studygolang.com/2012/12/go项目的目录结构/
