---
title: 33.ROS 配置Wireguard(VPN) 远程访问
author: Rayd62
date: 2022-11-28 14:16:06
tags:
  - ROS
  - 网络技术
---

# 配置 ROS

## 前置要求

1. ROS 本身是 Internet 出口
2. 出口已经开启端口转发，将 wireguard 所需端口映射到 ROS 上

满足任意条件即可

## 创建 Wireguard 接口

打开 winbox 连接 ROS，创建一个 wireguard 接口（配置保持默认即可）

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/2022-11-28-14:19:28-Pasted%20image%2020221006162403.png)

## 为 Wireguard 接口添加 IP 地址

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/2022-11-28-14:19:57-Pasted%20image%2020221006162516.png)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/2022-11-28-14:20:15-Pasted%20image%2020221006162611.png)

IP 地址配置为不与内网地址冲突的私网地址即可。

## 创建 Wireguard Peer

打开 `WireGuard` - `Peers` 点击创建。  
在对话框中，选择上面创建好的 `wireguard1` 接口  
`public key`：填写客户端配置文件中的公钥  
`allowed address`：为客户端连接后使用的 IP 地址，需要与 `wireguard1` 接口在同一网段。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/2022-11-28-14:20:28-Pasted%20image%2020221006162941.png)

## 为访问内网流量配置 NAT

打开`IP` - `Firewall`，选中`NAT`，点击添加。
`src address`：配置为`wireguard1` 网段
`out interface`：配置为 ROS 内网网段
`action`：配置为`masquerade`

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/2022-11-28-14:20:41-Pasted%20image%2020221006165610.png)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/2022-11-28-14:20:53-Pasted%20image%2020221006165702.png)

> 注意：一定要通过账号激活，否则性能不能完全打开。
