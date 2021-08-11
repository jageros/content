---
type: "posts"
author: "jager"
title: "Nginx命令操作记录"
date: "2021-08-06"
tags: [
"nginx",
]
---

+ 重新load配置 <br>
``nginx -s reload`` 
+ 热部署
> 先替换nginx二进制文件，然后发送``kill -USR2 进程号``，会启动新master
的进程<br>
> 然后向老进程发送信号让其优雅的关闭work进程 ``kill -WINCH 进程号`` , 老的master进程会保留防止回退