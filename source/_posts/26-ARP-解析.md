---
title: 26.ARP 解析
author: Rayd62
date: 2022-05-23 18:44:05
tags:
  - 网络技术
---
# ARP 作用
通过解析目的网络层地址来获取目的数据链路层地址的过程是由ARP（Address Resolution Protocol）协议来实现的。

在以太网中，希望仅通过IP 地址将数据包发送到目的地是不够的，因为IP 数据包一定要被封装成以太网帧才能在以太网中传送。ARP协议是TCP/IP协议簇中的重要组成部分，它能够通过目的
IP地址获取目标设备的MAC地址，从而实现数据链路层的可达性。

# ARP 构造
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203281408707.png)
+ 硬件类型（HTYPE）：如以太网（0x0001）、分组无线网。
+ 协议类型（PTYPE）：如网际协议(IP)（0x0800）、IPv6（0x86DD）。
+ 硬件地址长度（HLEN）：硬件地址的长度（单位字节），一般为6（以太网）。
+ 协议地址长度（PLEN）：协议地址（IP）的长度（单位字节），一般为4（IPv4）。
+ 操作码：1为ARP请求（request），2为ARP应答（reply），3为RARP请求，4为RARP应答。
+ 源硬件地址（Sender Hardware Address，简称SHA）：n个字节，n由硬件地址长度得到，以太网中为发送方MAC地址。
+ 源协议地址（Sender Protocol Address，简称SPA）：m个字节，m由协议地址长度得到，以太网中为发送方IP地址。
+ 目标硬件地址（Target Hardware Address，简称THA）：n个字节，n由硬件地址长度得到，以太网中为目标MAC地址。
+ 目标协议地址（Target Protocol Address，简称TPA）：m个字节，m由协议地址长度得到，以太网中为目标IP地址。

# ARP 工作过程
以主机A（10.0.0.1）向主机C（10.0.0.3）发送数据为例。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203281431367.png)

1. 当发送数据时，主机A会在自己的ARP缓存表中寻找是否有目标IP地址。如果找到就知道目标MAC地址为（00-01-02-03-04-CC），直接把目标MAC地址写入帧里面发送就可。
2. 本例中，主机A 的ARP 缓存表中不存在主机C 的MAC 地址，所以主机A 会发送ARP Request 来获取主机B 的MAC 地址。ARP Request 报文会被封装在以太网帧中，由于主机A 并不知道主机B 的MAC 地址，所以目的MAC 地址填入“FF-FF-FF-FF-FF-FF” ，这样ARP Request 报文就能够发送给广播域中的所有设备。
3. 主机A 会在三层报文中，填充目的IP（10.0.0.3）、源IP（10.0.0.1）、目的MAC（00-00-00-00-00-00）、源MAC（00-01-02-03-04- AA）、操作类型（Request - 代码1）
	![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203281551717.png)
4. 在所有广播域中的主机接收到主机A 发送的ARP Request 报文后，所有主机都会检查ARP 数据包中目的协议地址（IP 10.0.0.3）是否与自身网络层地址匹配。如果不匹配，则该主机将不会生成ARP Reply 回应（这里需要实验看一看，主机接到目的地址非本机的ARP Request 后，是否会将ARP reqeust 的源主机加入本机ARP 表）；如果匹配，则该主机会将ARP 报文中的源MAC 地址和源IP 地址信息记录到自己的ARP 缓存表中，然后通过ARP Reply 报文进行响应。
5. 主机C 会生成ARP Reply 报文回复主机A。ARP Reply 报文中目的MAC 为主机A 的MAC 地址（单播包），ARP 层填入自己的 MAC 地址到源MAC，操作类型为Reply（code 2）
	![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203281615004.png)
6. 主机A 收到ARP Reply 以后，会检查以太网帧中目的MAC 是否与自身MAC 匹配。如果匹配，ARP 报文中的源MAC 地址和源IP 地址会被记录到主机A 的ARP 缓存表中。


## ARP 操作代码
| 操作        | Code |
| ------------- | ---- |
| 请求-request  | 1    |
| 应答-reply | 2    |
| RARP 请求     | 3    |
| RARP 应答     | 4    |

## ARP 缓存
ARP 缓存表采用老化机制，在一段时间内如果表中某一条目没有被使用过，就会被自动删除。

# ARP 特殊使用场景
## ARP Proxy - ARP 代理
当一个节点代表另一个节点响应ARP请求时，就会发生代理ARP。

> **Proxy ARP occurs when one node is responding to an ARP request on behalf of another node.**

### Use Cases
#### 错误配置
如下图：
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203281938458.png)
假设现在有Host A与Host B 连接到Router C:
1. Host A IP addr 被错误配置成10.1.1.10/16，GW 为10.1.1.1
2. Host B IP addr IP 为10.1.4.20/24
3. Router C 连接Host A 一端为10.1.1.1/24；连接Host B 一端为10.1.4.1/24

在Host A 试图访问Host B 的过程中，由于Host A 配置错误，Host A 会将Host B 当作同网段主机处理，Host A 生成ARP Request 报文直接请求Host B 10.1.4.20 的MAC 地址而非其Gateway MAC 地址。

ARP Request 报文目的MAC 地址设置为"FF-FF-FF-FF-FF-FF" 会通告给网段内所有机器；但是Router C 正确配置，Router C 了解Eth0 和Eth1 属于不同网段也就是说Eth0 与Eth1 属于不同广播域，那么Router C不会将从Host A 收到的ARP Request 报文从Eth1 转发，这样Host A 无法获取Host B MAC 地址，也就无法与Host B 通信。

这时若Router C 配置了ARP 带答（默认情况下绝大多书路由器都不会开启ARP Proxy），Router C 可以以Eth0 的MAC 地址作为源MAC 生成ARP Reply 答复给Host A。那么Host A 收到 回复后会将Router C 的Eth0 MAC 地址"00-11-22-33-44-AB" 与Host B IP 地址"10.1.4.20"绑定写入ARP 缓存表中。

接下来Host A 将发送到Host B 的数据包将Router C Eth0 的MAC 地址作为目的MAC 地址封装成帧发送出去。根据目的MAC 地址这个以太网帧会发送到Router C 的Eth0 接口，接口检查目的MAC 地址是Eth0 本身，Router C 丢掉帧头将IP 数据包传给上层协议，上层协议检查目的地址为10.1.4.20，这时Router C 检查路由表判断由Router C 的Eth1 与目的网段直连，Router C 将IP 数据包从Eth1 接口封装发送出去。自此完成了Host A 到Host B 的数据传输。

在这个场景下，Host B 配置正确，Host B 判断发送到Host A 的回包目的地址与Host B 不属于同一网段，因此检查Host B 的Gateway MAC 地址是否存在与ARP 缓存表中，若不存在则发送ARP Request 请求Router C 的Eth1 MAC 地址；若存在则直接发送目的MAC 地址为Router C Eth1MAC 地址的数据包给Router C。

Router C 的Eth1 接收到来自Host B 的回包后，检查目的MAC 地址是Eth1 本身，Router C 丢掉帧头，将IP 数据包发送到上层协议，上层协议检查目的地址，由路由表判断需要通过Router C 的Eth0 转发数据包，则Router C 将IP 数据包封装好从Eth0 转发出去。Host A 自此收到Host B 的回包。

> 还有一种情况下也会发生这类ARP Proxy 的需求，即Host A 未配置网关Gateway。不推荐在这类配置错误的情况下通过ARP Proxy 解决通信问题。

#### NAT 下的ARP 代答
如下图：
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203282014143.png)
假设，现在有一个数据包A 从源IP **66.6.6.6** 希望发送给目的IP **72.2.2.10**。
此时，公司外部有一套防火墙，与ISP 的Router B 直连端口配置的是**72.2.2.2**；并且防火墙做了一条 DNAT 规则将所有发送给**72.2.2.10** 的数据包转发到内网的**10.1.1.45**。

数据包A 通过Internet 路由到Router B。Router B 收到数据包A 后检查目的地址为72.2.2.10 查询路由表后发现Router B 有一直连网段为72.2.2.0/24，这时Router B 判断可以将数据包A 直接发送给72.2.2.10 但是需要目的IP 为72.2.2.10 的MAC 地址。

Router B 查找自身ARP 表，若存在匹配条目，则直接生成以太网帧；若不存在匹配条目，则生成ARP Request 来查找IP 地址72.2.2.10 的MAC 地址是什么？

但是此时防火墙配置的接口地址为72.2.2.2/24，当防火墙收到ARP Request 后发现目的IP 的并非本机IP，则防火墙不响应这个ARP 请求。此时就需要NAT 设备使用ARP Proxy 代答，NAT 设备将72.2.2.2 接口的MAC 地址作为ARP Reply 的源MAC 回应Router B。Router B 收到回复后将IP 地址与MAC 地址写入ARP 表，并根据ARP 表声称以太网帧将数据传送给NAT 设备。NAT 设备触发DNAT 规则，将数据包转发给内网的服务器10.1.1.45，完成了整个数据包的传送过程。


> 在NAT设备上要求代理ARP的方法有多种，但它们涉及在上游路由器上使用静态路由来手动指示路由器发送必须转换到防火墙接口的数据包。虽然可行，但该解决方案不能扩展。


## Gratuitous ARP - 免费ARP
免费ARP是未收到ARP请求的ARP响应。免费ARP 作为节点通告或更新其到整个网络的IP到MAC映射的一种方式，免费ARP 是基于广播发送的。

### Use Case
#### 更新ARP 映射
第一个用例非常简单，如果节点的IP到MAC映射发生更改，节点可以使用免费的ARP来更新网络上其他主机的ARP映射。

如果用户手动修改其MAC地址，则可能会发生这种情况-他们保留相同的IP地址，但现在有了新的MAC地址。因此，必须更新与该用户通信的所有节点的ARP映射。（常见虚拟化环境）

#### 宣告节点的存在
当一台主机新加入一个网络时--它可以使用免费的ARP向网络宣布它的存在。尤其是当主机加入一个网络后希望抢先填充相邻主机的ARP缓存，而不需要它们启动传统的ARP过程。

但是并非所有主机都会缓存它收到的所有免费ARP 报文，因此这种方法也不太推荐。

> 需要与下面的ARP Probe 区分，ARP Probe 被用来探测IP 地址冲突的。

#### 冗余
当两个设备间冗余或故障切换时，使用免费ARP 是很实在的事情。一般来说这分为两种情况：
1. 两个设备共享IP 但每一台都有自己独立的MAC 地址
2. 两个设备共享IP 和MAC 地址

##### IP 地址冗余
在这种情况下，两台设备共享一个IP 地址，但每一台设备都有自己独立的MAC 地址。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203282102473.png)

在两台设备都正常工作时，一台设备Active 另一台Standy。Active 的设备宣告自己的MAC 地址"00-11-22-33-44-AA"和Shared IP 10.1.1.1 为一组ARP 映射。

但当Active 的设备故障时，Standby 设备通过使用免费ARP 将Shared IP 与自身的MAC 地址"00-11-22-33-44-BB" 通告给整个广播域，广播域中的所有设备收到ARP 后更新ARP 缓存表，接下来发送到10.1.1.1 的流量都会使用Standby 设备的MAC 地址"00-11-22-33-44-BB" 作为目的MAC 地址发送数据，而不是故障的Active 设备。

##### IP和MAC 冗余
在这种情况下，两台设备共享IP 地址和MAC 地址。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203282112263.png)
虽然这种情况下，主机的ARP映射永远不需要更新，但免费的ARP仍然至关重要。这时不是为了更新主机的ARP映射，而是为了更新下联交换机的MAC地址表，以便共享的MAC地址与正确的端口相关联。回想一下，交换机从任何接收到的帧的源MAC地址获取MAC地址映射。

> HSRP 和VRRP 都使用了免费ARP 的方法来进行故障切换。



## ARP Probe and ARP Announcement - ARP探针和ARP 通告
它们是ARP探测和ARP通告。这两种方法都在IP 地址冲突检测的过程中使用。

其想法是，如果主机获取并使用恰好已在网络上使用的IP地址，则会导致两台主机的连接问题。因此，主机在使用IP地址之前首先测试它，以确保它确实是唯一的，这是有益的。

该过程非常简单，发送几个ARP探测(通常为3个)，如果没有人响应，则使用ARP公告正式声明IP地址。

ARP Probe 和ARP Announcement 都使用广播包作为目的MAC 地址，因为要将ARP 数据包发送给广播域中的所有设备，以确定没有冲突的IP 地址。

# Reference
[Traditional ARP – Address Resolution Protocol – Practical Networking .net](https://www.practicalnetworking.net/series/arp/traditional-arp/)

[Proxy ARP – Definition and Use Cases – Practical Networking .net](https://www.practicalnetworking.net/series/arp/proxy-arp/)

[Gratuitous ARP – Definition and Use Cases – Practical Networking .net](https://www.practicalnetworking.net/series/arp/gratuitous-arp/)

[ARP Probe and ARP Announcement – Practical Networking .net](https://www.practicalnetworking.net/series/arp/arp-probe-arp-announcement/)