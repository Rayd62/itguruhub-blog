---
title: 25.HSRP - Hot Standby Router Protocol
author: Rayd62
date: 2022-05-23 18:14:10
tags:
  - 网络技术
---

# HSRP(Hot Standby Router Protocol)
在网络边界设备或接入链路上部署HSRP 可以迅速地、无感知地避免用户发生第一跳故障（first-hop failure ）的情况。同样属于FHRP(First Hop Redundancy Protocol) 的协议还有Virtual Router Redundancy Protocol (VRRP) 和Gateway Load Balancing Protocol(GLBP).

# HSRP 的背景
## 为什么是HSRP 来保障主机访问其它网络的弹性？
虽然主机上有多种可以不熟的动态路由器发现机制（Dynamic Router Discovery Mechanism），但这些机制大多都不会帮助网络管理员提供他所需要的网络弹性（故障恢复）。因为DRDM 开发的初衷并非是为主机提供网络弹性；同时还需要注意的是，绝大多数的主机在配置是只允许管理员配置一个默认网关（default gateway）。

### Proxy ARP（ARP 代答）
部署ARP 代答的设备，在访问其它网络时会通过发送ARP 包请求远端主机的IP 对应的MAC 地址。而连接了该网络的网络设备（路由器或三层交换机）会将自己的MAC 地址作为 ARP 回复发送给主机，这样主机就能将数据包交给正确的网关。如果此时这台网络设备故障，那么主机必须等待本机ARP 表老化，重发ARP 请求远端主机IP 对应的MAC 地址，并得到另一网关设备的正确答复，才能恢复通信。（win 10, win srv 2012 的默认ARP 老化时间为30s）

### Dynamic Routing Protocol （动态路由协议）
从管理、操作、安全等等角度来看，在所有主机上运行动态路由协议来避免网关丢失（第一跳故障）是不可行的。
同时，并非所有的主机都能支持动态路由协议。

### ICMP Router DIscovery Protocol （ICMP 路由器发现协议）
一些主机上部署有ICMP Router Discovery Protocol（IRDP， [RFC 1256](http://www.faqs.org/rfcs/rfc1256.html)），该协议能够在路由失效时为主机寻找新的网关设备。一个部署了IRDP 的主机会侦听来来自路由器（路由器需要做相应的配置）的Hello 组播包，当主机不再收到来自路由器的Hello 组播包时能切换另一台路由器作为新的网关设备。默认Advertisement 速率为每7到10分钟一次，默认生存期为30分钟。这决定了使用 IRDP 不适合作为网关设备故障探测的手段。

### Dynamic Host Configuration Protocol （DHCP）
在TCP/IP 网络中，DHCP 为主机提供网络配置下发的能力。但DHCP 并不具备在网关设备故障时，向已有配置的主机重新下发新的默认网关的能力。

### HSRP 是怎么解决网关失效的问题？
使用HSRP，一组（两个或以上）网关设备（路由器/交换机）协同工作，向局域网上的主机呈现单个虚拟路由器的假象。
组中的单台网关设备被选举来负责转发所有来自子网内主机发送到虚拟路由器的流量。这台网关设备被称为活动路由器（Active Router）。其它组内的网关设备都被选举为备用路由器（Standby Router）。当活动路由器故障时，备用路由器承担活动路由器的转发职责。需要注意的是，即使存在多台网关设备，只有活动路由器能够转发流量。

# HSRP 的工作模式
## 术语
`Active Rotuer`: 指当前为虚拟路由器转发流量的活动路由器

`Standby Router`: 指主备用路由器

`Standby Group`: 参与模拟虚拟路由器的一组路由器

`Hello Time`: 来自给定路由器的连续HSRP Hello消息之间的间隔

`Hold Time`: 从收到Hello消息到假定发送路由器出现故障之间的时间间隔

## HSRP 选举
HSRP 有两个主要角色（Role）Active 和Standby。路由器是否能进入某个角色，需要`Standby Group`的所有路由器选举得到。

在HSRP 选举的过程中，有两个参数尤为重要：
1. HSRP Priority
2. 参与选举的接口IP 地址

在选举过程中，HSRP Priority 大的路由器选举成功成为Active Router。如果HSRP Priority 值一致，那么接口IP 地址大的会选举成功成为Active Router。

默认情况下，HSRP 的Priority 值为100，可用取值范围为0-255.

## HSRP 故障切换（角色切换）
### 接口故障
当故障发生时HSRP 的收敛时间取决于`Hello Time 计时器`和`Hold Time 计时器`。

默认情况下，两个计时器的时间分别设置为3s 和10s。这意味着每3s `HSRP Group`的设备之间会发送一次`Hello Message`，并且如果10s 内未收到`Hello Message`，则`Standby Router` 会切换成`Active Router`。

> to avoid increased CPU usage and unnecessary standby state flapping, do not set the hello timer below one (1) second or the hold timer below 4 seconds.

当有超过2个路由器加入组时，选举只会产生1个`Active Router` 和1个`Standby Router`，其余路由器会保持`Listen` 状态；在`Active Router` 失效，`Standby Router` 状态转变为`Active Router` 后，其余路由器会重新进入`Speak` 状态来选举新的`Standby Router`。

### 配合HSRP Tracking Mechanism 功能发生的抢占行为
如果使用了`HSRP Tracking Mechanism` ，并且追踪的行为生效，则故障切换或者抢占行为会立即发生，无论此时的两个计时器状态如何。

### HSRP Timer
| Timer         | Description                                                                                                                                                                                                                                 |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Active Timer  | This timer is used to monitor the active router. This timer starts any time an active router receives a hello packet. This timer expires in accordance with the hold time value that is set in the related field of the HSRP hello message. |
| Standby Timer | This timer is used in order to monitor the standby router. The timer starts any time the standby router receives a hello packet. This timer expires in accordance with the hold time value that is set in the respective hello packet.      |
| Hello timer   | This timer is used to clock hello packets. All HSRP routers in any HSRP state generate a hello packet when this hello timer expires.                                                                                                        |



## HSRP Message
### HSRP v1
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202205191228016.png)

## HSRP 地址
### HSRP v1 地址
业务IP（子网GW）：虚拟IP一个，子网内其它地方不能使用该IP。
Hello Message dst IP：224.0.0.2
Mac Address：0000.0c07.acXX(X 指可替换值，因此在HSRP v1 中，一个网络设备最多支持255 个Standby Group，这是MAC 的限制)
UDP：1985


### HSRP v2 地址
业务IP（子网GW）：虚拟IP一个，子网内其它地方不能使用该IP。
Hello Message dst IP：224.0.0.102
Mac Address：0000.0c07.aXXX(X 指可替换值，因此在HSRP v2 中，一个网络设备最多支持4095 个Standby Group，但是虚拟MAC 的取值范围尚未找到相关文档支持)
UDP：1985

## HSRP 状态
### Initial
HSRP 尚未运行，当接口状态UP 或配置变更时会进入此状态。

### Learn
路由器尚未决定VIP，也并没有收到信任的`Active Router` 发来的 `Hello Message`。这个状态下，路由器会等待`Active Router` 的信息。

### Listen
路由器获取到VIP，但既不是`Active Router` 也不是`Standby Router`。能够从组播地址收到`Hello Message`。

### Speak
路由器发送周期性的`Hello Message`来主动参加HSRP 的active/standby 选举。只有配置了VIP 的路由器才能进入`Speak` 状态。

### Standby
该路由器是下一个活动路由器的候选路由器，并定期发送Hello消息。

### Active
路由器当前正在转发发送到组的虚拟MAC地址的数据包。

## HSRP v1 VS. HSRP v2
| Item                                        | HSRP v1       | HSRP v2                                           |
| ------------------------------------------- | ------------- | ------------------------------------------------- |
| milisecond timer                            | 不支持        | 支持                                              |
| group numbers                               | 0-255         | 0-4095                                            |
| physical device identified by hello message | 不支持        | 支持，Hello Message 6bytes 字段记录发送者MAC 地址 |
| Multicase address                           | 224.0.0.2（此地址） | 224.0.0.102                                                  |

# HSRP Feature
## HSRP 抢占 - Preemption
HSRP 抢占机制允许拥有高HSRP 优先级的网络设备立即成为`Active Router`。
当一个高优先级的网络设备从低优先级的网络设备抢占`Active Router`，会发送一条报文通告`Standby Group`。
当低优先级网络设备收到从高优先级设备发出的抢占报文，网络设备会改变状态到`Speak`，然后`Standby Group` 开始重新选举`Standby Router`。


## HSRP 抢占延迟 - Preempt Delay
抢占延迟机制允许路由器延后xx 秒抢占`Active Router`，此机制主要是为了让路由器有充足的时间来完成相应的路由收敛。

该机制从发生一次抢占开始，例如Router A的出口链路故障，引发自身HSRP Priority 下降，被Router B 抢占了`Active Router`的角色。此时Delay 计时器开始计时？（还是说该台设备链路恢复，Priority 重新提高，等待一定delay 时间等待路由收敛）

## 端口/链路检测 - Interface Tracking
端口/链路检测允许管理员在网络设备上指定HSRP进程的监控另一个接口的状态，以便更改HSRP 的HSRP Priority 值。

使用场景：如果指定接口的Line Protocol DOWN，则此路由器的HSRP优先级会根据配置自动调低，从而允许让`Standby Group` 中的另一个优先级更高的网络设备抢占`Active Router`。

>如果HSRP 配置监控了多个端口，那么HSRP Priority 的值调整是累加的。例如，配置一个端口故障，Priority 降低10；另一个端口故障，Priority 降低5，那么当两个端口都故障的时候，HSRP Priority 的值会下降15.

## 使用网络设备MAC 地址 - Use Burned-In Address
使用Burned-In Address 特性可以让`HSRP Group` 使用设备接口的MAC 地址为用户提供服务。

>实施`Use-bia`命令是为了克服在令牌环接口上使用HSRP MAC地址的功能地址的限制。

但是使用`use-bia` 命令会有以下几点劣势：
1. 当一个新的网络设备成为`Active Router` 时，`Virtual IP address` 会与新的网络设备的接口地址绑定，这时新的网络设备会发送`gratuitous ARP` 通告内网用户， ARP 映射关系发生变化，但是并非所有内网主机都能识别`gratuitous ARP` 数据包的；
2. `Proxy ARP` 会被`use-bia` 命令破坏其功能性。成为`Active Router` 的网络设备会无法恢复已有的ARP 数据库；
3. 如果配置了`use-bis`，则只允许一个HSRP组。

## Multiple HSRP Group
此功能进一步实现了网络内的冗余和负载共享，并允许更充分地利用冗余路由器。当路由器主动转发一个HSRP组的流量时，它可以处于备用状态，也可以处于另一个组的侦听状态。

## 配置HSRP Group MAC 地址
通常，您使用HSRP来帮助终端站定位IP路由的第一跳网关。终端站配置有默认网关。但是，HSRP可以为其他协议提供第一跳冗余。某些协议(例如高级对等网络(APPN))使用MAC地址来标识第一跳以实现路由目的。

在这种情况下，通常需要能够使用备用mac-Address命令指定虚拟MAC地址。虚拟IP地址对于这些协议并不重要。该命令的实际语法为`STANDBY [GROUP] MAC-ADDRESS Mac-Address`。

## HSRP with ICMP Redicret 
每个子网上运行两个(或更多)HSRP组，配置的HSRP组至少与参与的路由器数量一样多。配置优先级时，每台路由器都是至少一个HSRP组的主路由器。当一台路由器确定将终端站重定向到特定目的地的另一台路由器时，它不是将终端站重定向到另一台路由器的IP地址，而是找到将该路由器作为其主路由器的HSRP组，并将终端站重定向到相应的虚拟IP地址。如果该目标路由器随后出现故障，HSRP将确保另一台路由器接管其工作，并可能将终端站重定向到另一台虚拟路由器。

# Reference
[What is HSRP ? | Hot Standby Router Protocol ⋆ IpCisco](https://ipcisco.com/lesson/hsrp/)

[Hot Standby Router Protocol Features and Functionality - Cisco](https://www.cisco.com/c/en/us/support/docs/ip/hot-standby-router-protocol-hsrp/9234-hsrpguidetoc.html#hrspadd)

[RFC 2281: Cisco Hot Standby Router Protocol (HSRP) (rfc-editor.org)](https://www.rfc-editor.org/rfc/rfc2281.html)

[Hot Standby Router Protocol (HSRP): Frequently Asked Questions - Cisco](https://www.cisco.com/c/en/us/support/docs/ip/hot-standby-router-protocol-hsrp/9281-3.html)