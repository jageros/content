---
type: "posts"
author: "jager"
title: "go服务端开发学习指南"
date: "2021-08-03"
tags: [
"go",
"服务端",
"学习指南",
]
---

> 本文介绍基于go语言的服务端程序开发学习指南，根据列举的知识点自行学习，所列知识点都是开发基础必备技术栈。
<!--more-->

## **熟练Linux系统的使用**
+ Linux的基本命令操作
  ``必须熟练Linux系统的使用，才能得心应手的部署自己开发的服务端程序。``
+ shell脚本的编写
  ``编写一下自动化脚本，用于控制服务器的一些自动操作，如起停服，备份，批量操作文件等等。``
+ 系统其他配置等
  ``熟悉Linux系统和一些常用软件的配置等，才能让系统更加适配自己的服务端程序的运行，即运行环境的配置。``
+ make和Makefile等
  ``makefile用于配置程序的编译法则，make即编译程序``
  ps：基础学习可以参考：[菜鸟教程](https://www.runoob.com/linux/linux-tutorial.html)

## **语言基础**
+ golang的语法学习： [golang菜鸟教程](https://www.runoob.com/go/go-tutorial.html)

## **数据结构与算法**
+ 队列，大顶堆，小顶堆，红黑树，字典树
+ 八大排序算法，雪花算法等
+ 缓存的过期策略：FIFO，LRU，LFU等

## **数据库**
+ Redis
  ``用于做缓存,队列,主题订阅发布等``

+ MongoDB
  ``结构化不强的数据存储``
+ MySQL
  ``结构化数据存储``
+ etcd
  ``服务注册与发现``
+ Elasticsearch
  ``搜索，数据分析``


## **开发框架**
+ gin
+ go-kit
+ go-zero
+ kratos
+ beego
+ gRpc
+ zap

## **架构知识**
+ 分布式
+ 微服务

## **其他知识点**
+ rpc
+ protobuf
+ http
+ TCP
+ websocket
+ Docker
+ Kubernetes

## **[golang工程师学习之路](https://time.geekbang.org/learning/path-detail/11)**