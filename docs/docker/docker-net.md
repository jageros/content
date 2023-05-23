---
type: "posts"
author: "jager"
title: "Docker网络"
date: "2023-05-23"
tags: [
"docker",
]
---

> Docker 网络是一个用于管理容器之间通信的关键组件。每个容器都可以连接到一个或多个 Docker 网络，并且可以使用网络内的名称进行相互通信。Docker 提供了多种网络驱动程序，每个驱动程序都有不同的特性和适用场景。

<!--more-->

## 目录

1. Docker 网络概述
2. Docker 网络驱动程序
    - 默认网络驱动程序：bridge
    - host 网络驱动程序
    - none 网络驱动程序
    - overlay 网络驱动程序
    - macvlan 网络驱动程序
3. 创建 Docker 网络
4. 连接容器到网络
5. 容器间网络通信
6. 连接容器到外部网络
7. 网络别名和链接
8. 清理和删除网络

### 1. Docker 网络概述

Docker 网络是一个用于管理容器之间通信的关键组件。每个容器都可以连接到一个或多个 Docker 网络，并且可以使用网络内的名称进行相互通信。Docker 提供了多种网络驱动程序，每个驱动程序都有不同的特性和适用场景。

### 2. Docker 网络驱动程序

Docker 提供了以下几种网络驱动程序：

- **bridge**：默认的网络驱动程序，创建一个连接到主机物理网络的桥接接口。
- **host**：容器和主机共享网络栈，容器直接使用主机的网络接口。
- **none**：容器没有网络连接，只有回环接口。
- **overlay**：用于跨多个 Docker 守护进程创建的容器的网络，允许不同主机上的容器通信。
- **macvlan**：将容器连接到现有物理网络，容器使用自己的 MAC 地址。

### 3. 创建 Docker 网络

使用 Docker 命令行工具可以创建 Docker 网络。以下是创建网络的示例命令：

```
$ docker network create mynetwork
```

此命令将创建一个名为 `mynetwork` 的新网络。

### 4. 连接容器到网络

要将容器连接到一个或多个网络，可以在创建容器时使用 `--network` 标志。例如：

```
$ docker run --network=mynetwork --name=mycontainer nginx
```

这将在 `mynetwork` 网络中创建一个名为 `mycontainer` 的容器。

### 5. 容器间网络通信

当容器连接到同一个网络时，它们可以使用容器名称进行相互通信。例如，在网络中的一个容器可以通过另一个容器的名称来访问该容器。例如：

```
$ docker exec -it container1 ping container2
```

这将在 `container1` 容器中执行一个命令，向 `container2` 发送

一个 ping 请求。

### 6. 连接容器到外部网络

要将容器连接到外部网络，可以使用 `--network` 标志和 `--publish` 标志。例如：

```
$ docker run --network=bridge --publish 8080:80 nginx
```

这将在默认的 `bridge` 网络中创建一个容器，并将容器的 80 端口映射到主机的 8080 端口。

### 7. 网络别名和链接

Docker 允许为容器设置网络别名，以便更方便地进行容器间通信。使用 `--network-alias` 标志来设置网络别名。例如：

```
$ docker run --network=mynetwork --network-alias=myalias nginx
```

此命令将在 `mynetwork` 网络中创建一个名为 `myalias` 的别名。

另外，使用 `--link` 标志可以链接一个容器到另一个容器。例如：

```
$ docker run --name=container1 --link=container2 nginx
```

这将创建一个名为 `container1` 的容器，并链接到名为 `container2` 的容器。

### 8. 清理和删除网络

要清理不再需要的网络，可以使用以下命令：

```
$ docker network rm mynetwork
```

这将删除名为 `mynetwork` 的网络。