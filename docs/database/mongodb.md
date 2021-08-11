---
type: "posts"
author: "jager"
title: "MongoDB的一些操作记录"
date: "2021-08-06"
tags: ["mongodb"]
---

> 本文记录MongoDB使用过程中，因需求需要进行一些不常用的操作，如创建用户，导出数据，数据备份恢复等运维层面的技术知识，
> 后续有遇到继续往文章后面添加，以备不时之需，可来此进行查阅

<!--more-->

+ MongoDB创建用户
```shell
# 用户名xxx, 密码123, 角色权限：readWrite(读写), 数据库：xxx
db.createUser({user:'xxx', pwd:'123', roles:[{ role: "readWrite", db: "xxx" }]})
```

+ MongoDB导出数据
```shell
# 数据库：xxxdb, 用户名：xxx, 密码：123, 表名：players, -f 需要导出的字段...  --type=csv(导出格式) -o ./data.csv(导出路径)
mongoexport -d xxxdb -u xxx -p 123 -c players -f _id,data.ip,data.createTime,data.lastLoginTime,data.resource.1,data.resource.2,data.resource.4,data.VideoTotal,data.withdraw.withdrawTotal --type=csv -o ./data.csv

# 导出的Excel表中时间戳转成时间字符串
=TEXT((C2+8*3600)/86400+70*365+19,"yyyy-mm-dd hh:mm:ss")
```
  
+ MongoDB备份数据
```shell
# 数据库名： xxxdb, 用户名：xxx, 密码： 123, 备份文件路径：/root/workspace/mongo/backup
mongodump -d xxxdb -o /root/workspace/mongo/backup -u xxx -p 123
```