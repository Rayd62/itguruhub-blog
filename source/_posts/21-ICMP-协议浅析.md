---
title: 21.ICMP 协议浅析
author: Rayd62
date: 2022-05-02 18:24:11
tags:
  - 网络技术
---

# ICMP
ICMP: Internet Control Message Protocol
互联网控制信息协议是IP 层的一个辅助协议。用来在网络设备传递各种差错和控制信息。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202205011652691.png)

常用的ICMP 类型
| Type | Code | Description  |
| ---- | ---- | ------------ |
| 0    | 0    | Echo Reply   |
| 3    | 0    | 网络不可达   |
| 3    | 1    | 主机不可达   |
| 3    | 2    | 协议不可达   |
| 3    | 3    | 端口不可达   |
| 5    | 0    | 重定向       |
| 8    | 0    | Echo Request |
 
# ICMP 重定向
ICMP重定向报文是ICMP 控制报文的一种。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202205011706161.png)

| Type | Code | Desc                                                     |
| ---- | ---- | -------------------------------------------------------- |
| 5    | 0    | redirect datagrames for the Network                      |
| 5    | 1    | redirect datagrames for the Host                         |
| 5    | 2    | redirect datagrames for the Type of Service and Networks |
| 5    | 3    | redirect datagrames for the Type of Service and Host     |

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202205011655195.png)
Steps:
1. 主机A 10.0.0.1/24希望前往服务器A 20.0.0.1/24，因为在不同网段，主机A 将数据包发送到默认网关RTB
2. 网关RTB 查询路由表后发现，目标地址20.0.0.1 的下一跳在RTA 10.0.0.200，这时RTB 监测到RTA 与数据包的源地址10.0.0.1 在同一网段，RTB 会向主机A 发送ICMP 重定向报文，告知主机A 去往20.0.0.1/24 的流量可以直接发送到RTA，因为这是前往服务器A 的最短路径
3. 主机A 接下来发往服务器A 的流量都直接发往同网段的RTA

> 对于具有IP源路由选项和目的地地址字段中的网关地址的数据报，即使存在比源路由中的下一个地址更好的到达最终目的地的路由，也不会发送重定向消息。

## 抓包
主机A 发往默认网关10.0.0.1 的数据包无回复；
默认网关发送ICMP Redirect 到主机A，告知主机A 去往20.0.0.2 的数据包直接发往10.0.0.200
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202205031734118.png)

主机A 重新发送去往20.0.0.2 的数据包，这次下一条用的是10.0.0.200，可以从二层的MAC 地址看出；
同时这次发送的request 包，收到正常的reply 回包。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202205031736585.png)

## 发送ICMP redirect 条件
满足下列**所有条件**，则路由器会发送ICMP redirect 信息：
1. 数据包进入路由器的接口与数据包路由出的接口相同。
2. 源IP地址的子网或网络与所路由的数据包的下一跳IP地址位于同一子网或网络上。
3. 该数据报不携带源路由信息。
4. 内核配置为发送重定向。


# ICMP 错误报告
ICMP 定义了各种错误消息，用户诊断网络连接性问题；根据这些错误消息，源设备可以判断出数据传输丢失的原因。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202205011911554.png)

| Type | Code | Desc                                             |
| ---- | ---- | ------------------------------------------------ |
| 3    | 0    | net unreachable, send by gateway                 |
| 3    | 1    | host unreachable, send by gateway                |
| 3    | 2    | protocol unreachable, send by destination host   |
| 3    | 3    | port unreachable, send by destination host       | 
| 3    | 4    | fragmentation needed and DF set, send by gateway |
| 3    | 5    | source route failed, send by gateway             |

## net unreachable
根据网关路由信息，目的网络不可达时，网关会向源主机发送`destination unreachable` 信息。

## protocol unreachable
如果目标主机的IP 模块无法正常工作，则目标主机会向源主机回复`protocol unreachable`

## fragmentation needed and DF set
如果网络设备转发报文需要对报文分片，而IP 的DF 位设置为1，则网关丢弃报文，同时发送`fragmentation needed and DF set` 信息给源主机。

# ICMP 超时报文
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202205011920370.png)

| Type | Code | Desc                              |
| ---- | ---- | --------------------------------- |
| 11   | 0    | time to live exceeded in transit  |
| 11   | 1    | fragment reassembly time exceeded |

## TTL 传输超时
如果在网络设备转发报文时，TTL 字段为0，则网络设备丢弃报文同时向源设备发送ICMP 超时信息 `time to live exceeded in transit`

## IP 重组超时
如果正在重组分段数据报的主机由于在其时间限制内丢失片段而无法完成重组，则它将丢弃该数据报，并且它可能发送`fragment reassembly time exceeded`消息。

# 源抑制消息 - Source Quench Message
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202205012216241.png)

| Type | Code | Desc                  |
| ---- | ---- | --------------------- |
| 4    | 0    | Source Quench Message |

1. 互联网线路上的网络设备若没有足够的buffer 来存储需要转发的数据包，那么在丢弃数据包前，网络设备可以向源设备发送源抑制消息。
2. 目标设备若收到太多数据包而无法处理，可以向源设备发送源抑制消息。

收到源抑制消息的源主机应该降低向指定目的地发送流量的速率，直到它不再从网络或目标主机接收到源抑制消息。然后，源主机可以逐渐提高向目的地发送流量的速率，直到再次收到源抑制消息。
