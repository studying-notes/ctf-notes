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

## 使用Zenmap进行扫描

Zenmap是安全扫描工具Nmap的一个官方的图形用户界面，是一个跨平台的开源应用，不仅方便初学者使用，同时为高级使用者提供了很多高级特性。频繁的扫描能够被存储，进行重复运行。命令行工具提供了直接与Nmap的交互操作。扫描结果能够被存储以便于事后查阅。存储的扫描可以被比较，以辨别其异同。

## Ping扫描

在Nmap中提供了很多扫描方式，其中就有Ping扫描方式，Ping扫描只进行Ping，然后显示出在线的主机。扫描时只需要加入-sP选项就可以很方便地启用Ping扫描，使用该选项的时候，Nmap仅进行Ping扫描，然后回显出做出响应的主机，使用该选项扫描可以轻易地获取目标信息而不会被轻易发现。在默认的情况下，Nmap会发送一个ICMP回声请求和一个TCP报文到目标端口。Ping扫描的优点是不会返回太多的信息造成对结果的分析，并且这是一种非常高效的扫描方式。

```shell
nmap -sP 192.168.4.171/24
```

## 无Ping扫描

无Ping扫描通常用于防火墙禁止Ping的情况下，它能确定正在运行的机器。默认情况下，Nmap只对正在运行的主机进行高强度的探测，如端口扫描、版本探测或者操作系统探测。用-P0禁止主机发现会使Nmap对每一个指定的目标IP地址进行所要求的扫描，这可以穿透防火墙，也可以避免被防火墙发现。需要注意的是，-P0的第二个字符是数字0而不是字母O。使用“nmap -P0【协议1、协议2】【目标】”进行扫描。

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

无Ping扫描也可以躲避某些防火墙的防护，可以在目标主机禁止Ping的情况下使用。

## TCP SYN Ping扫描

TCP 协议是 TCP/IP 协议族中的面向连接的、可靠的传输层协议，允许发送和接收字节流形式的数据。为了使服务器和客户端以不同的速度产生和消费数据，TCP 提供了发送和接收两个缓冲区。TCP 提供全双工服务，数据同时能双向流动。通信的每一方都有发送和接收两个缓冲区，可以双向发送数据。TCP 在报文中加上一个递进的确认序列号来告诉发送者，接收者期望收到的下一个字节，如果在规定时间内，没有收到关于这个包的确认响应，则重新发送此包，这保证了 TCP 是一种可靠的传输层协议。

-PS 选项发送一个设置了 SYN 标志位的空 TCP 报文。默认目的端口为 80（可以通过改变 nmap.h）文件中的 DEFAULT-TCP-PROBE-PORT 值进行配置，但不同的端口也可以作为选项指定，甚至可以指定一个以逗号分隔的端口列表（如 -PS22，23，25，80，115，3306，3389），在这种情况下，每个端口会被并发地扫描。

通常情况下，Nmap 默认 Ping 扫描是使用 TCP ACK 和 ICMP Echo 请求对目标进行是否存活的响应，当目标主机的防火墙阻止这些请求时，我们可以使用 TCP SYN Ping 扫描来进行对目标主机存活的判断。

```shell
nmap -PS -v 192.168.4.171/24
```

从上面的返回结果可得知Nmap是通过SYN/ACK和RST响应来对目标主机是否存活进行判断，但在特定情况下防火墙会丢弃RST包，这种情况下扫描的结果会不准确，这时，我们需要指定一个端口或端口范围来避免这种情况。

```shell
nmap -PS'80,100-200' -v 192.168.4.171/24
```

## TCP ACK Ping扫描

使用 -PA 选项可以进行 TCP ACK Ping 扫描，它与 TCP SYN Ping 扫描是非常类似的，唯一的区别是设置 TCP 的标志位是 ACK 而不是 SYN，使用这种方式扫描可以探测阻止 SYN 包或 ICMP Echo 请求的主机。

很多防火墙会封锁 SYN 报文，所以 Nmap 提供了 TCP SYN Ping 扫描与 TCP ACK Ping 扫描两种探测方式，这两种方式可以极大地提高通过防火墙的概率，我们还可以同时使用 -PS 与 -SA 来既发送 SYN 又发送 ACK。在使用 TCP ACK Ping 扫描时，Nmap 会发送一个 ACK 标志的 TCP 包给目标主机，如果目标主机不是存活状态则不响应该请求，如果目标主机在线则会返回一个 RST 包。

```shell
nmap -PA -v 192.168.4.171/24
```

同时使用-PS与-PA选项，代码如下。

```shell
nmap -PA -PS -v 192.168.4.171/24
```

## 被防火墙阻止的案例

首先使用TCP ACK Ping方式对目标主机进行扫描。

从输出的结果中发现目标主机是没有存活的状态，尝试使用TCP SYN Ping进行扫描，对目标主机的存活状态进行判断。

从输出的结果得知目标主机是存活状态，由此可以说明TCP ACK包被目标主机防火墙阻止了。

## UDP Ping扫描

-PU选项是发送一个空的UDP报文到指定端口。如果不指定端口则默认是40125。该默认值可以通过在编译时改变nmap.h文件中的 DEFAULT-UDP-PROBE-PORT值进行配置。默认使用这样一个奇怪的端口是因为对于开放端口，很少会使用这种扫描方式。

使用UDP Ping扫描时Nmap会发送一个空的UDP包到目标主机，如果目标主机响应则返回一个ICMP端口不可达错误，如果目标主机不是存活状态则会返回各种ICMP错误信息。

```shell
nmap -PU -v 192.168.121.1
```

从输出的结果中可以得知目标主机是存活状态，这表明目标主机存活时会返回一个ICMP端口不可达的信息，这里我们使用Wireshark获取数据包分析。

如图2.4所示，捕获的包中有一条信息为“Destination unreachable”，这表明目标不可达，在详细信息面板中可以看到使用的端口为40125，也可以指定使用其他端口。

## ICMP Ping Types扫描

使用 -PE ； -PP ； -PM 选项可以进行 ICMP Ping Types 扫描。ICMP（Internet Control Message Protocol）是 Internet 控制报文协议。它是 TCP/IP 协议族的一个子协议，用于在 IP 主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。这些控制消息虽然并不传输用户数据，但是对于用户数据的传递起着重要的作用。

Nmap 发送一个 ICMP type 8（回声请求）报文到目标 IP 地址，从运行的主机得到一个 type 0（回声响应）报文。-PE 选项简单地来说是通过向目标发送 ICMP Echo 数据包来探测目标主机是否在线，正因为许多主机的防火墙会禁止这些报文，所以仅仅 ICMP 扫描对于互联网上的目标通常是不够的。但对于系统管理员监视一个内部网络，它们可能是实际有效的途径。使用 -PE 选项打开该回声请求功能。-PP 选项是 ICMP 时间戳 Ping 扫描，虽然大多数的防火墙配置不允许 ICMP Echo 请求，但由于配置不当可能回复 ICMP 时间戳请求，所以可以使用 ICMP 时间戳来确定目标主机是否存活。-PM 选项可以进行 ICMP 地址掩码 Ping 扫描。这种扫描方式会试图用备选的 ICMP 等级 Ping 指定主机，通常有不错的穿透防火墙的效果。

（1）使用ICMP Echo扫描方式

```shell
nmap -PE -v 192.168.121.1
```

（2）使用ICMP时间戳Ping扫描

```shell
nmap -PP -v 163.com
```

（3）使用ICMP地址掩码Ping扫描

```shell
nmap -PM -v 192.168.121.1
```

可以看到，不同的扫描方式穿过不同的防火墙时有着不同的结果。

## ARP Ping扫描

-PR选项通常在扫描局域网时使用。地址解析协议，即ARP（Address Resolution Protocol），是根据IP地址获取物理地址的一个TCP/IP协议，其功能是：主机将ARP请求广播到网络上的所有主机，并接收返回消息，确定目标IP地址的物理地址，同时将IP地址和硬件地址存入本机ARP缓存中，下次请求时直接查询ARP缓存。

ARP Ping扫描是Nmap对目标进行一个ARP Ping的过程，尤其在内网的情况下，使用ARP Ping扫描方式是最有效的，在本地局域网中防火墙不会禁止ARP请求，这就使得它比其他Ping扫描都更加高效，在内网中使用ARP Ping是非常有效的。在默认情况下，如果Nmap发现目标主机就在它所在的局域网上，会进行ARP扫描。即使指定了不同的Ping类型（如-PI或者–PS），Nmap也会对任何相同局域网上的目标机使用ARP。如果不想使用ARP扫描，可以指定--send-ip。

```shell
nmap -PR 192.168.0.1
```

## 扫描列表

列表扫描是主机发现的退化形式，它仅仅列出指定网络上的每台主机，不发送任何报文到目标主机。默认情况下，Nmap仍然对主机进行反向域名解析以获取它们的名字。

```shell
nmap -sL 192.168.0.1/24
```

## 禁止反向域名解析

```shell
nmap -n -sL 192.168.0.1/24
```

-n选项意为禁止解析域名，使用该选项的时候Nmap永远不对目标IP地址作反向域名解析。

该选项很少使用，如果是对一台有域名绑定的服务器通常不会使用该选项；如果是单纯扫描一段IP，使用该选项可以大幅度减少目标主机的相应时间，从而更快地得到结果。

## 反向域名解析

-R选项意为反向解析域名，使用该选项时Nmap永远对目标IP地址作反向域名解析。

```shell
nmap -R -sL 192.168.0.1/24
```

## 使用系统域名解析器

--system-dns意为使用系统域名解析器。默认情况下，Nmap通过直接发送查询到您主机上配置的域名服务器来解析域名。为了提高性能，许多请求（一般几十个）并发执行。如果您希望使用系统自带的解析器，就指定该选项（通过getnameinfo()调用一次解析一个IP）。

```shell
nmap --system-dns 192.168.0.106
```

## 扫描一个IPv6地址

IPv6是Internet Protocol Version 6的缩写，其中，Internet Protocol译为“互联网协议”。IPv6是IETF（Internet Engineering Task Force，互联网工程任务组）设计的用于替代现行版本IP协议（IPv4）的下一代IP协议。目前IP协议的版本号是4（简称为IPv4），它的下一个版本就是IPv6。

Nmap很早就支持对IPv6的扫描，我们在Nmap选项中使用-6选项就可以进行对IPv6的扫描。

```shell
nmap -6 240e:390:4b0:1c81:e65f:1ff:fe04:f489
```

## 路由跟踪

使用--traceroute选项即可进行路由跟踪，使用路由跟踪功能可以帮助用户了解网络的同行情况，通过此选项可以轻松地查出从本地计算机到目标之间所经过的网络节点，并可以看到通过各个节点的时间。

```shell
nmap --traceroute -v www.163.com
```

## SCTP INIT Ping扫描

SCTP（Stream Control Transmission Protocol，流控制传输协议）是IETF（Internet Engineering Task Force，因特网工程任务组）在2000年定义的一个传输层（Transport Layer）协议。SCTP可以看作是TCP协议的改进，它改进了TCP的一些不足，SCTP INIT Ping扫描通过向目标发送INIT包，根据目标主机的相应判断目标主机是否存活。

```shell
nmap -PY -v 192.168.0.106
```
