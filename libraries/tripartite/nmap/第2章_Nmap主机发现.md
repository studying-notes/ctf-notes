---
date: 2022-04-02T15:15:37+08:00
author: "Rustle Karl"

title: "第2章_Nmap主机发现"
url:  "posts/ctf/libraries/tripartite/nmap/第2章_Nmap主机发现"  # 永久链接
tags: ["CTF", "README"] # 标签
series: ["CTF 学习笔记"] # 系列
categories: ["学习笔记"] # 分类

toc: true # 目录
draft: false # 草稿
---

## 选项列表

- `-sP` Ping 扫描
- `-P0` 无 Ping 扫描
- `-PS` TCP SYN Ping 扫描
- `-PA` TCP ACK Ping 扫描
- `-PU` UDP Ping 扫描
- `-PE;-PP;-PM` ICMP Ping Types 扫描
- `-PR` ARP Ping 扫描
- `-n` 禁止DNS反向解析
- `-R` 反向解析域名
- `-system-dns` 使用系统域名解析器
- `-sL` 列表扫描
- `-6` 扫描 IPv6 地址
- `-traceroute` 路由跟踪
- `-PY` SCTP INIT Ping 扫描

## 一次简单的扫描

```shell
$ nmap 192.168.4.171

Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-02 16:04 中国标准时间
Nmap scan report for host.docker.internal (192.168.4.171)
Host is up (0.00017s latency).
Not shown: 990 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
902/tcp  open  iss-realsecure
912/tcp  open  apex-mesh
1027/tcp open  IIS
2179/tcp open  vmrdp
3389/tcp open  ms-wbt-server
5357/tcp open  wsdapi

Nmap done: 1 IP address (1 host up) scanned in 0.73 seconds
```

从以上的扫描结果中可以很轻易发现开放（open）的端口，在端口后我们也可以发现相关的服务名称，后面的章节会详细介绍。

## 使用 Zenmap 进行扫描

Zenmap 是安全扫描工具 Nmap 的一个官方的图形用户界面，是一个跨平台的开源应用，不仅方便初学者使用，同时为高级使用者提供了很多高级特性。频繁的扫描能够被存储，进行重复运行。命令行工具提供了直接与 Nmap 的交互操作。扫描结果能够被存储以便于事后查阅。存储的扫描可以被比较，以辨别其异同。

## Ping 扫描

在 Nmap 中提供了很多扫描方式，其中就有 Ping 扫描方式，Ping 扫描只进行 Ping，然后显示出在线的主机。扫描时只需要加入 -sP 选项就可以很方便地启用 Ping 扫描，使用该选项的时候，Nmap 仅进行 Ping 扫描，然后回显出做出响应的主机，使用该选项扫描可以轻易地获取目标信息而不会被轻易发现。在默认的情况下，Nmap 会发送一个 ICMP 回声请求和一个 TCP 报文到目标端口。Ping 扫描的优点是不会返回太多的信息造成对结果的分析，并且这是一种非常高效的扫描方式。

```shell
nmap -sP 192.168.4.171/24
```

## 无 Ping 扫描

无 Ping 扫描通常用于防火墙禁止 Ping 的情况下，它能确定正在运行的机器。默认情况下，Nmap 只对正在运行的主机进行高强度的探测，如端口扫描、版本探测或者操作系统探测。用 -P0 禁止主机发现会使 Nmap 对每一个指定的目标 IP 地址进行所要求的扫描，这可以穿透防火墙，也可以避免被防火墙发现。需要注意的是，-P0 的第二个字符是数字 0 而不是字母 O。使用“nmap -P0 【协议 1、协议 2 】【目标】”进行扫描。

```shell
nmap -P0 192.168.4.171/24
```

如果没有指定任何协议，Nmap 会默认使用协议 1、协议 2、协议 4，如果想知道这些协议是如何判断目标主机是否存活可以使用 --packet-trace 选项。

我们也可以手动指定扫描目标主机的协议，Nmap 支持的协议和编号如下所示：

- ① TCP：对应协议编号为6。
- ② ICMP：对应协议编号为1。
- ③ IGMP：对应协议编号为2。
- ④ UDP：对应协议编号为17。

我们指定使用 TCP、UDP、IGMP 协议向目标主机发送包并判断目标主机是否在线。

```shell
nmap -p'06,17,2' --packet-trace 192.168.4.171
```

无 Ping 扫描也可以躲避某些防火墙的防护，可以在目标主机禁止 Ping 的情况下使用。

## TCP SYN Ping 扫描

TCP 协议是 TCP/IP 协议族中的面向连接的、可靠的传输层协议，允许发送和接收字节流形式的数据。为了使服务器和客户端以不同的速度产生和消费数据，TCP 提供了发送和接收两个缓冲区。TCP 提供全双工服务，数据同时能双向流动。通信的每一方都有发送和接收两个缓冲区，可以双向发送数据。TCP 在报文中加上一个递进的确认序列号来告诉发送者，接收者期望收到的下一个字节，如果在规定时间内，没有收到关于这个包的确认响应，则重新发送此包，这保证了 TCP 是一种可靠的传输层协议。

`-PS` 选项发送一个设置了 SYN 标志位的空 TCP 报文。默认目的端口为 80（可以通过改变 nmap.h）文件中的 DEFAULT-TCP-PROBE-PORT 值进行配置，但不同的端口也可以作为选项指定，甚至可以指定一个以逗号分隔的端口列表（如 `-PS22,23,25,80,115,3306,3389`），在这种情况下，每个端口会被并发地扫描。

通常情况下，Nmap 默认 Ping 扫描是使用 TCP ACK 和 ICMP Echo 请求对目标进行是否存活的响应，当目标主机的防火墙阻止这些请求时，我们可以使用 TCP SYN Ping 扫描来进行对目标主机存活的判断。

```shell
nmap -PS -v 192.168.4.171/24
```

从上面的返回结果可得知 Nmap 是**通过 SYN/ACK 和 RST 响应来对目标主机是否存活进行判断**，但在**特定情况下防火墙会丢弃 RST 包**，这种情况下扫描的结果会不准确，这时，我们需要指定一个端口或端口范围来避免这种情况。

```shell
nmap -PS'80,100-200' -v 192.168.4.171/24
```

## TCP ACK Ping 扫描

使用 -PA 选项可以进行 TCP ACK Ping 扫描，它与 TCP SYN Ping 扫描是非常类似的，唯一的区别是设置 TCP 的标志位是 ACK 而不是 SYN，使用这种方式扫描可以探测阻止 SYN 包或 ICMP Echo 请求的主机。

很多防火墙会封锁 SYN 报文，所以 Nmap 提供了 TCP SYN Ping 扫描与 TCP ACK Ping 扫描两种探测方式，这两种方式可以极大地提高通过防火墙的概率，我们还可以同时使用 -PS 与 -SA 来既发送 SYN 又发送 ACK。在使用 TCP ACK Ping 扫描时，Nmap 会发送一个 ACK 标志的 TCP 包给目标主机，如果目标主机不是存活状态则不响应该请求，如果目标主机在线则会返回一个 RST 包。

```shell
nmap -PA -v 192.168.4.171/24
```

同时使用 -PS 与 -PA 选项，代码如下。

```shell
nmap -PA -PS -v 192.168.4.171/24
```

## 被防火墙阻止的案例

首先使用 TCP ACK Ping 方式对目标主机进行扫描。

从输出的结果中发现目标主机是没有存活的状态，尝试使用 TCP SYN Ping 进行扫描，对目标主机的存活状态进行判断。

从输出的结果得知目标主机是存活状态，由此可以说明 TCP ACK 包被目标主机防火墙阻止了。

## UDP Ping扫描

`-PU` 选项是发送一个空的 UDP 报文到指定端口。如果不指定端口则默认是 40125。该默认值可以通过在编译时改变 nmap.h 文件中的 DEFAULT-UDP-PROBE-PORT 值进行配置。默认使用这样一个奇怪的端口是因为对于开放端口，很少会使用这种扫描方式。

使用 UDP Ping 扫描时 Nmap 会发送一个空的 UDP 包到目标主机，如果目标主机响应则返回一个 ICMP 端口不可达错误，如果目标主机不是存活状态则会返回各种 ICMP 错误信息。

```shell
nmap -PU -v 192.168.121.1
```

从输出的结果中可以得知目标主机是存活状态，这表明目标主机存活时会返回一个 ICMP 端口不可达的信息，这里我们使用 Wireshark 获取数据包分析。

如图 2.4 所示，捕获的包中有一条信息为“Destination unreachable”，这表明目标不可达，在详细信息面板中可以看到使用的端口为 40125，也可以指定使用其他端口。

## ICMP Ping Types扫描

使用 -PE ； -PP ； -PM 选项可以进行 ICMP Ping Types 扫描。ICMP（Internet Control Message Protocol）是 Internet 控制报文协议。它是 TCP/IP 协议族的一个子协议，用于在 IP 主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。这些控制消息虽然并不传输用户数据，但是对于用户数据的传递起着重要的作用。

Nmap 发送一个 ICMP type 8（回声请求）报文到目标 IP 地址，从运行的主机得到一个 type 0（回声响应）报文。-PE 选项简单地来说是通过向目标发送 ICMP Echo 数据包来探测目标主机是否在线，正因为许多主机的防火墙会禁止这些报文，所以仅仅 ICMP 扫描对于互联网上的目标通常是不够的。但对于系统管理员监视一个内部网络，它们可能是实际有效的途径。使用 -PE 选项打开该回声请求功能。-PP 选项是 ICMP 时间戳 Ping 扫描，虽然大多数的防火墙配置不允许 ICMP Echo 请求，但由于配置不当可能回复 ICMP 时间戳请求，所以可以使用 ICMP 时间戳来确定目标主机是否存活。-PM 选项可以进行 ICMP 地址掩码 Ping 扫描。这种扫描方式会试图用备选的 ICMP 等级 Ping 指定主机，通常有不错的穿透防火墙的效果。

### 使用 ICMP Echo 扫描方式

```shell
nmap -PE -v 192.168.121.1
```

### 使用 ICMP 时间戳 Ping 扫描

```shell
nmap -PP -v 163.com
```

### 使用 ICMP 地址掩码 Ping 扫描

```shell
nmap -PM -v 192.168.121.1
```

可以看到，不同的扫描方式穿过不同的防火墙时有着不同的结果。

## ARP Ping扫描

`-PR` 选项通常在扫描局域网时使用。地址解析协议，即ARP（Address Resolution Protocol），是根据IP地址获取物理地址的一个TCP/IP 协议，其功能是：主机将 ARP 请求广播到网络上的所有主机，并接收返回消息，确定目标 IP 地址的物理地址，同时将 IP 地址和硬件地址存入本机 ARP 缓存中，下次请求时直接查询 ARP 缓存。

ARP Ping 扫描是 Nmap 对目标进行一个 ARP Ping 的过程，尤其在内网的情况下，使用 ARP Ping 扫描方式是最有效的，在本地局域网中防火墙不会禁止 ARP 请求，这就使得它比其他 Ping 扫描都更加高效，在内网中使用 ARP Ping 是非常有效的。在默认情况下，如果 Nmap 发现目标主机就在它所在的局域网上，会进行 ARP 扫描。即使指定了不同的 Ping 类型（如 -PI 或者– PS），Nmap 也会对任何相同局域网上的目标机使用 ARP。如果不想使用 ARP 扫描，可以指定 --send-ip。

```shell
nmap -PR 192.168.0.1
```

```shell
nmap -PR 192.168.0.1/24
```

## 扫描列表

列表扫描是主机发现的退化形式，它仅仅列出指定网络上的每台主机，不发送任何报文到目标主机。默认情况下，Nmap 仍然对主机进行反向域名解析以获取它们的名字。

```shell
nmap -sL 192.168.0.1/24
```

## 禁止反向域名解析

```shell
nmap -n -sL 192.168.0.1/24
```

-n 选项意为禁止解析域名，使用该选项的时候 Nmap 永远不对目标 IP 地址作反向域名解析。

该选项很少使用，如果是对一台有域名绑定的服务器通常不会使用该选项；如果是单纯扫描一段 IP，使用该选项可以大幅度减少目标主机的相应时间，从而更快地得到结果。

## 反向域名解析

-R 选项意为反向解析域名，使用该选项时 Nmap 永远对目标 IP 地址作反向域名解析。

```shell
nmap -R -sL 192.168.0.1/24
```

## 使用系统域名解析器

`--system-dns` 意为使用系统域名解析器。默认情况下，Nmap 通过直接发送查询到您主机上配置的域名服务器来解析域名。为了提高性能，许多请求（一般几十个）并发执行。如果您希望使用系统自带的解析器，就指定该选项（通过 `getnameinfo()` 调用一次解析一个 IP）。

```shell
nmap --system-dns 192.168.0.106
```

## 扫描一个 IPv6 地址

IPv6 是 Internet Protocol Version 6 的缩写，其中，Internet Protocol 译为“互联网协议”。IPv6 是 IETF（Internet Engineering Task Force，互联网工程任务组）设计的用于替代现行版本 IP 协议（IPv4）的下一代 IP 协议。目前 IP 协议的版本号是 4（简称为 IPv4），它的下一个版本就是 IPv6。

Nmap 很早就支持对 IPv6 的扫描，我们在 Nmap 选项中使用 -6 选项就可以进行对 IPv6 的扫描。

```shell
nmap -6 240e:390:4b0:1c81:e65f:1ff:fe04:f489
```

## 路由跟踪

使用 --traceroute 选项即可进行路由跟踪，使用路由跟踪功能可以帮助用户了解网络的同行情况，通过此选项可以轻松地查出从本地计算机到目标之间所经过的网络节点，并可以看到通过各个节点的时间。

```shell
nmap --traceroute -v www.163.com
```

## SCTP INIT Ping 扫描

SCTP（Stream Control Transmission Protocol，流控制传输协议）是 IETF（Internet Engineering Task Force，因特网工程任务组）在 2000 年定义的一个传输层（Transport Layer）协议。SCTP 可以看作是 TCP 协议的改进，它改进了 TCP 的一些不足，SCTP INIT Ping 扫描通过向目标发送 INIT 包，根据目标主机的相应判断目标主机是否存活。

```shell
nmap -PY -v 192.168.0.106
```
