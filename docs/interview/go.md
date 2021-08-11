---
type: "posts"
author: "jager"
title: "Go语言面试题"
date: "2021-08-06"
tags: [
"面试问题",
]
draft: true
---

> 面试知识点记录
<!--more-->

## **go语言知识点**
### *golang底层是怎么管理调度协程的？*
1. 每个用户线程对应多个内核空间线程，同时也可以一个内核空间线程对应多个用户空间线程。Go采用这种模型，使用多个内核线程管理多个goroutine。
2. GPM调度模型.
3. h

### *C++和golang比较。*

### gin
1. Gin的中间件是如何实现的
2. Gin的一次请求整个过程是如何调用的
3. Gin的基数树知道吗，说一下
4. Gin请求里的request buffer可以读几次