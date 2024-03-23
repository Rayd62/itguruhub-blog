---
title: 47.firewalld.richlanguage
author: Rayd62
date: 2023-05-26 22:36:11
tags:
  - Linux
  - 安全
---

# firewalld.richlanguage

## 一般的 Rule 结构

```bash
General rule structure

           rule
             [source]
             [destination]
             service|port|protocol|icmp-block|icmp-type|masquerade|forward-port|source-port
             [log]
             [audit]
             [accept|reject|drop|mark]
```

| 参数         | 值                                                                                                                                                                                              | remark                                                                                                                                                                 |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| rule         | rule [family="ipv4\|ipv6"] [priority=-32768~32767]                                                                                                                                              | priority 越小越优先，负数会比所有原规则优先执行，正数会在所有原规则之后执行                                                                                            |
| source       | source [not] address="address[/mask]"\|mac="mac-address"\|ipset="ipset"                                                                                                                         |                                                                                                                                                                        |
| destination  | destination [not] address="address[/mask]"                                                                                                                                                      |                                                                                                                                                                        |
| service      | service name="service name"                                                                                                                                                                     |                                                                                                                                                                        |
| port         | port port="port value" protocol="tcp\|udp"                                                                                                                                                      | port value 可以是一个也可以是一个 x-y 的范围值                                                                                                                         |
| protocol     | protocol value="protocal value"                                                                                                                                                                 | 可以是 protocol id 或名称，请在/etc/protocols 中查看可用值                                                                                                             |
| ICMP-Block   | icmp-block name="icmptype name"                                                                                                                                                                 | 可以通过 firewall-cmd —get-icmptypes 查看可用值，使用该参数，是无法指定 action 的，默认为 reject                                                                       |
| Masquerade   | masquerade                                                                                                                                                                                      | IP forwarding 会被隐含启用，无法指定 action                                                                                                                            |
| ICMP-Type    | icmp-type name="icmptype name"                                                                                                                                                                  | 同 ICMP-Block 但此处可以设置 action                                                                                                                                    |
| Forward-Port | forward-port port="port value" protocol="tcp\|udp" to-port="port value" to-addr="address"                                                                                                       | 可以将本地某端口接到的流量转发给本地其他端口、其他机器或其他机器其他端口                                                                                               |
| Source-Port  | source-port port="port value" protocol="tcp\|udp"                                                                                                                                               | port value 可以是一个也可以是一个 x-y 的范围值                                                                                                                         |
| Log          | log [prefix="prefix text"] [level="emerg\|alert\|crit\|error\|warning\|notice\|info\|debug"][limit value="rate/duration"]                                                                       |                                                                                                                                                                        |
| Audit        | audit [limit value="rate/duration"]                                                                                                                                                             | another way to logging using audit records sent to the service auditd                                                                                                  |
| Action       | accept [limit value="rate/duration"] \| reject [type="reject type] [limit value="rate/duration"] \| drop [limit value="rate/duration"] \| mark set="mark[/mask]" [limite value="rate/duration"] | 带有标记的所有数据包将在标记表中的标记和掩码组合中标记在 mangle 表的 PREROUTING 链中。For valid reject types see --reject-with type in iptables-extensions(8) man page |

## Examples

These are examples of how to specify rich language rules. This format (i.e. one string that specifies whole rule) uses for example firewall-cmd --add-rich-rule (see firewall-cmd(1)) as well as D-Bus interface.

- Example 1 Enable new IPv4 and IPv6 connections for protocol 'ah'

```
rule protocol value="ah" accept
```

- Example 2 Allow new IPv4 and IPv6 connections for service ftp and log 1 per minute using audit

```
rule service name="ftp" log limit value="1/m" audit accept
```

- Example 3 Allow new IPv4 connections from address 192.168.0.0/24 for service tftp and log 1 per minutes using syslog

```
rule family="ipv4" source address="192.168.0.0/24" service name="tftp" log prefix="tftp" level="info" limit value="1/m" accept
```

- Example 4 New IPv6 connections from 1:2:3:4:6:: to service radius are all rejected and logged at a rate of 3 per minute. New IPv6 connections from other sources are accepted.

```
rule family="ipv6" source address="1:2:3:4:6::" service name="radius" log prefix="dns" level="info" limit value="3/m" reject
rule family="ipv6" service name="radius" accept
```

- Example 5 Forward IPv6 port/packets receiving from 1:2:3:4:6:: on port 4011 with protocol tcp to 1::2:3:4:7 on port 4012

```bash
rule family="ipv6" source address="1:2:3:4:6::" forward-port to-addr="1::2:3:4:7" to-port="4012" protocol="tcp" port="4011"
```

- Example 6 White-list source address to allow all connections from 192.168.2.2

```
rule family="ipv4" source address="192.168.2.2" accept
```

- Example 7 Black-list source address to reject all connections from 192.168.2.3

```
rule family="ipv4" source address="192.168.2.3" reject type="icmp-admin-prohibited"
```

- Example 8 Black-list source address to drop all connections from 192.168.2.4

```
rule family="ipv4" source address="192.168.2.4" drop
```
