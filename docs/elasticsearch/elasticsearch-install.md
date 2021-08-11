---
type: "posts"
author: "jager"
title: "Elasticsearch部署"
date: "2021-08-03"
tags: [
"elasticsearch",
"搜索引擎",
]
---

> Elasticsearch是一个开源的分布式、RESTful 风格的搜索和数据分析引擎，它的底层是开源库Apache Lucene。
> Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库——无论是开源还是私有，但它也仅仅只是一个库。
> 本文记录Linux上Elasticsearch的安装配置等部署流程

<!--more-->

## 下载Elasticsearch

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.9.2-linux-x86_64.tar.gz
```

## 创建用户

```
groupadd esGroup
useradd user -g esGroup -p password
```

## 权限分配

```
cd /usr/local
chown -R esUser:esGroup elasticsearch-7.1.0

vim /etc/security/limits.conf
*  soft  nofile  65536
*  hard  nofile  65536
```

## 修改系统配置

```
vim /etc/sysctl.conf
vm.max_map_count=262144
sysctl -p
```

## 修改elasticsearch配置(重点关注7项)

```
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
cluster.name: yourscat-es-cluster   //-------------(1)---------------------
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: node-1   //-------------(2)---------------------
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /usr/local/elasticsearch-7.1.0/data  //-------------(3)---------------------
#
# Path to log files:
#
path.logs: /usr/local/elasticsearch-7.1.0/logs  //-------------(4)---------------------
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 0.0.0.0  //-------------(5)---------------------
#
# Set a custom port for HTTP:
#
http.port: 9200   //-------------(6)---------------------
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["node-1"]  //-------------(7)---------------------
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true

```

## 启动elasticsearch

```
bin/elasticsearch -d
```

## 部署elasticHD

```
yum install xdg-utils
```
