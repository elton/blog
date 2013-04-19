---
layout: post
title: '让Golang利用多核CPU能力来计算π的值'
description: 'golang, pi'
date: 2013-04-19
comments: true
categories:
- Go
tags:
- Go
- routines
- pi
- 并行计算

---
使用go的routines和channel，可以充分利用多核处理器，提高高CPU资源占用计算的速度。如下列计算π的值

```
package main

import (
	"fmt"
	"runtime"
	"time"
)

var n int64 = 10000000000
var h float64 = 1.0 / float64(n)

func f(a float64) float64 {
	return 4.0 / (1.0 + a*a)
}

func chunk(start, end int64, c chan float64) {
	var sum float64 = 0.0
	for i := start; i < end; i++ {
		x := h * (float64(i) + 0.5)
		sum += f(x)
	}
	c <- sum * h
}

func main() {

	//记录开始时间
	start := time.Now()

	var pi float64
	np := runtime.NumCPU()
	runtime.GOMAXPROCS(np)
	c := make(chan float64, np)

	for i := 0; i < np; i++ {
		go chunk(int64(i)*n/int64(np), (int64(i)+1)*n/int64(np), c)
	}

	for i := 0; i < np; i++ {
		pi += <-c
	}

	fmt.Println("Pi: ", pi)

	//记录结束时间
	end := time.Now()

	//输出执行时间，单位为毫秒。
	fmt.Printf("spend time: %vs\n", end.Sub(start).Seconds())
}
```

在我的2.6 GHz Intel Core i74核处理器下，Mac 10.8.3系统，运行上述程序

```
$./pi 
Pi:  3.141592653589691
spend time: 29.779854372s
```

执行过程中，cpu占用400%，说明已经充分利用现有CPU的处理性能。 可以看到用Go来进行并行计算还是比较方便的。