---
date: 2022-04-04T13:52:53+08:00
author: "Rustle Karl"

title: "第11章_Nmap技巧"
url:  "posts/ctf/libraries/tripartite/nmap/第11章_Nmap技巧"  # 永久链接
tags: ["CTF", "README"] # 标签
series: ["CTF 学习笔记"] # 系列
categories: ["学习笔记"] # 分类

toc: true # 目录
draft: false # 草稿
---

## 功能选项

- --send-eth 发送以太网数据包
- --send-ip 网络层发送
- --privileged 假定拥有所有权
- --interactive在交互模式中启动
- -V 查看Nmap版本号
- -d 设置调试级别
- --packet-trace 跟踪发送接受的报文
- --iftist 列举接口和路由
- -e 指定网络接口
- -oG 继续中断扫描
- firewalk 探测防火墙
- vmauthd-brute VMWare认证破解

## 发送以太网数据包

--send-eth 选项用于发送以太网数据包，该选项会要求 Nmap 在数据链路层发送报文，而不是在网络层发送报文。需要注意的是，在 UNIX 中无论是否使用该选项，Nmap 都会使用原 IP 包。

## VMWare认证破解

Nmap中的vmauthd-brute脚本可以破解安装虚拟机系统的用户名与密码。

```shell
nmap -p 902 --script vmauthd-brute 192.168.121.1
```

```shell

```


## 二级

### 三级

```shell

```

```shell

```


## 二级

### 三级

```shell

```

```shell

```


## 二级

### 三级

```shell

```

```shell

```


## 二级

### 三级

```shell

```

```shell

```


