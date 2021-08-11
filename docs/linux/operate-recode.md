---
type: "posts"
author: "jager"
title: "Linux不常用操作记录"
date: "2021-08-03"
tags: [
"linux",
"命令行",
]
---

> 本文记录在使用Linux系统过程中，遇到的一些不常用的，需要查询的，难以记忆的操作命令，
> 方便以后再次使用的时候，可以翻阅查询，提高开发效率，提高记忆

<!--more-->

## 批量重命名图片

```shell
let n=0; for i in *.jpg; do mv $i $n.jpg; n=`expr $n + 1`; done
```

## 递归删除指定类型文件

```shell
find . -name "*.rej" |xargs rm -f
```

## 批量修改文件指定内容

将文件中的gopkg.in/mgo.v2/bson修改成go.mongodb.org/mongo-driver/bson

```shell
# --include=*.go 表示指定文件类型
grep gopkg.in/mgo.v2/bson -rl --include=*.go ./* | xargs sed -i "s#gopkg.in/mgo.v2/bson#go.mongodb.org/mongo-driver/bson#g"
# 或
sed -i "s#gopkg.in/mgo.v2/bson#go.mongodb.org/mongo-driver/bson#g" `grep gopkg.in/mgo.v2/bson -rl --include="*.go" ./`
```

## 查看端口占用

```shell
lsof -i:8080

# 或
netstat -pan | grep 8080

#netstat 中参数选项
#-a或--all：显示所有连线中的Socket； 
#-A<网络类型>或--<网络类型>：列出该网络类型连线中的相关地址； 
#-c或--continuous：持续列出网络状态； 
#-C或--cache：显示路由器配置的快取信息； 
#-e或--extend：显示网络其他相关信息； 
#-F或--fib：显示FIB； 
#-g或--groups：显示多重广播功能群组组员名单； 
#-h或--help：在线帮助； 
#-i或--interfaces：显示网络界面信息表单； 
#-l或--listening：显示监控中的服务器的Socket； 
#-M或--masquerade：显示伪装的网络连线； 
#-n或--numeric：直接使用ip地址，而不通过域名服务器； 
#-N或--netlink或--symbolic：显示网络硬件外围设备的符号连接名称； 
#-o或--timers：显示计时器； 
#-p或--programs：显示正在使用Socket的程序识别码和程序名称； 
#-r或--route：显示Routing Table； 
#-s或--statistice：显示网络工作信息统计表； 
#-t或--tcp：显示TCP传输协议的连线状况； 
#-u或--udp：显示UDP传输协议的连线状况； 
#-v或--verbose：显示指令执行过程； 
#-V或--version：显示版本信息； 
#-w或--raw：显示RAW传输协议的连线状况； 
#-x或--unix：此参数的效果和指定"-A unix"参数相同； 
#--ip或--inet：此参数的效果和指定"-A inet"参数相同。
```

## 修改终端显示信息

> 配置文件： /etc/bashrc

```
PS1=”[\u@\h \w]$”

\d ：代表日期，格式为weekday month date，例如：”Mon Aug 1”

\H ：完整的主机名称。例如：我的机器名称为：fc4.linux，则这个名称就是fc4.linux

\h ：仅取主机的第一个名字，如上例，则为fc4，.linux则被省略

\t ：显示时间为24小时格式，如：HH：MM：SS

\T ：显示时间为12小时格式

\A ：显示时间为24小时格式：HH：MM

\u ：当前用户的账号名称

\v ：BASH的版本信息

\w ：完整的工作目录名称。家目录会以 ~代替

\W ：利用basename取得工作目录名称，所以只会列出最后一个目录

# ：下达的第几个命令

$ ：提示字符，如果是root时，提示符为：# ，普通用户则为：$
```

## 查看系统参数

```shell
# 查看Linux版本
cat /proc/version

# 查看Linux发行版本
cat /etc/redhat-release

# 查看CPU型号及参数
cat /proc/cpuinfo | grep model

# 查看磁盘空间
df -h

# 查看内存和交换空间
free -[k/m/g]     #k/m/g分别表示以kb，mb，gb为单位显示``

# 查看系统位数
getconf LONG_BIT
```

## linux文件修改权限
```
命令: chmod ABC file

其中A、B、C各为一个数字，分别表示User、Group、及Other的权限。

A、B、C这三个数字如果各自转换成由“0”、“1”组成的二进制数，则二进制数的每一位分别代表一个角色的读、写、运行的权限。比如User组的权限A：

如果可读、可写、可运行，就表示为二进制的111，转换成十进制就是7。
如果可读、可写、不可运行，就表示为二进制的110，转换成十进制就是6。
如果可读、不可写、可运行，就表示为二进制的101，转换成十进制就是5。


“4=r,2=w,1=x”的意思是：
r 代表读，w 代表写，x 代表执行，
如果可读，权限是二进制的100，十进制是4；
如果可写，权限是二进制的010，十进制是2；
如果可运行，权限是二进制的001，十进制是1；

具备多个权限，就把相应的 4、2、1 相加就可以了：
若要 rwx 则 4+2+1=7
若要 rw- 则 4+2=6
若要 r-x 则 4+1=5
若要 r-- 则 =4
若要 -wx 则 2+1=3
若要 -w- 则 =2
若要 --x 则 =1
若要 --- 则 =0

为不同的角色分配不同的权限，放在一起，就出现 777、677这样的数字了。
你也可以用 chmod u+x file 的方式为User组添加执行权限。
chmod +x file  是为当前用户添加执行权限
```