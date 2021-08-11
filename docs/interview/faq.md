---
type: "posts"
author: "jager"
title: "面试灵魂拷问"
date: "2021-08-06"
tags: [
"面试问题",
]
---

> 本文收集一些面试golang服务端开发的问题和答案，以及一些知识点的参考文章等，让即将面试的小伙伴有佛脚可抱。
> 包括但不限于Linux， golang， 数据库， nginx， etcd， Redis， http， 消息队列，等等。

<!--more-->

## 问答集
>Q1: Linux的磁盘管理 <br>
_A_:

>Q2: Linux有哪些进程通信方式，五大件 <br>
A:

>Q3: Linux的共享内存如何实现，大概说了一下 <br>
A:

>Q4: 共享内存实现的具体步骤 <br>

>Q5: socket网络编程，说一下TCP的三次握手和四次挥手 <br>

>Q6:如何把docker讲的很清楚 <br>

>Q7:cgroup在linux的具体实现 <br>

>Q8:多线程用过哪些 <br>

>Q9:MySQL索引的实现，innodb的索引，b+树索引是怎么实现的，为什么用b+树做索引节点，一个节点存了多少数据，怎么规定大小，与磁盘页对应 <br>

>Q10:MySQL的事务隔离级别，分别解决什么问题 <br>

>Q11:Redis了解么，如果Redis有1亿个key，使用keys命令是否会影响线上服务，我说会，因为是单线程模型，可以部署多个节点。 <br>

>Q12:知不知道有一条命令可以实现上面这个功能 <br>

>Q13:Redis的持久化方式，aod和rdb，具体怎么实现，追加日志和备份文件，底层实现原理 <br>

>Q14:Redis的list是怎么实现的，我说用ziplist+quicklist实现的，ziplist压缩空间，quicklist实现链表 <br>

>Q15:sortedset怎么实现的，使用dict+skiplist实现的，问我skiplist的数据结构，大概说了下是个实现简单的快速查询结构。 <br>

>Q16:了解什么消息队列，rmq和kafka <br>

>Q17:线程池 <br>

>Q18:操作系统的进程通信方式，僵尸进程和孤儿进程是什么，如何避免僵尸进程，我说让父进程显示通知，那父进程怎么知道子进程结束了 <br>

>Q19:计算机网络TCP和UDP有什么区别，为什么迅雷下载是基于UDP的，我说FTP是基于TCP，而迅雷是p2p不需要TCP那么可靠的传输保证。 <br>

>Q20:操作系统的死锁必要条件，如何避免死锁 <br>

>Q21:写一个LRU的缓存，需要完成超时淘汰和LRU淘汰。 <br>

>Q22: get和post的区别？ <br>
A： https://blog.csdn.net/kebi007/article/details/103059900

>Q23: Redis 问题集 <br>
A： https://blog.csdn.net/Design407/article/details/103242874

> Q24: 消息中间件面试题31道RabbitMQ+ActiveMQ+Kafka <br>
A: https://www.jianshu.com/p/0a322b2bba2a
