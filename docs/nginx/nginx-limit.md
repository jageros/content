---
type: "posts"
author: "jager"
title: "Nginx IP 限流"
date: "2021-08-03"
tags: [
"nginx",
"限流",
]
---

> 通过对同一IP进行限流，在一定程度上可以防止应用层DDOS攻击。本文介绍 Nginx对同一IP限流的配置

<!--more-->

## nginx.conf配置文件
```
http {
	......
	# 在http节点中添加下面这一行
	limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
	server {
		# 在server节点中添加这一行
		limit_req zone=one burst=10 nodelay;;
		......
	}
}
```


limit_req_zone的参数详解:

``binary_remote_addr``：表示客户端ip地址的二进制，当此nginx前方还存在代理时，需进行处理.

``zone=one:10m``：为session会话状态分配一个大小为10m内存存储区，并且设定名称为one.

``rate=1r/s``：表示请求频率不能超过每秒一次.

---

limit_req的参数详解:

``zone=one``：表示这个server服务收到的请求放在one的那个内存区域，等待被处理.

``burst=10``：表示请求队列的长度，比如我设置了rate=1r/s，而同一时刻有15个请求发过来，那么第一个请求会被处理，10个请求会被存在队列里等待被处理，而剩下4个请求就会直接响应503状态码。

``nodelay``：表示不延时。比如rate=1r/s，如果不设置nodelay就会严格按照1秒处理一个请求的频率，直观的看就是页面数据卡了，过了一秒后才加载出来。

> 这里有人就会疑惑了，既然nodelay设置不延时，那么设置处理频率rate=1r/s又有什么意义呢?
> 
> 我们假设一个案例来做分析：<br>
> >设置rate=1r/s，burst=10，nodelay不延时，然后同时有15个请求发过来，其中一个请求被处理的时候，有10个请求会在队列中等待，而另外4个请求就会直接响应503状态码。由于设置了nodelay，一旦处理完一个请求之后立刻就会处理队列中的下一个请求。在1秒钟内，队列钟的请求全部都被处理完了，但是这个时候队列中的请求并不会被清除掉，而是会等到满1秒钟后才会被清除。这期间如果还有请求发来，由于队列中的请求并没有被清除，所以直接就响应503。
