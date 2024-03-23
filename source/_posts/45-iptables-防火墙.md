---
title: 45.iptables 防火墙
author: Rayd62
date: 2023-05-26 22:27:33
tags:
  - Linux
  - 安全
---

# iptables 防火墙

## 什么是 iptables

iptables 是一个配置**Linux 内核防火墙**的命令行工具。iptables 也经常指代内核级防火墙。

iptables 可以检测、修改、转发、重定向和丢弃 IPv4 数据包。

过滤 IPv4 数据包的代码已内置于内核中，并且按照不同的目的被组织成**表**的集合，每一个**表**都有**不同的目的**。**表**由一组预先定义的**链**组成，**链**包含依序遍历的**规则**。每条规则由可条件和在条件匹配为真时执行的动作（被称为 target）组成。如果一个数据包通过了一个链的所有匹配（如果这个链是空的，也被视为通过所有匹配），链的策略决定了这个数据包的最终去向。

iptables 就是允许管理员来操作链/规则的用户工具

iptables 是基于 Netfilter 框架的，所以所有的网络相关操作都可以由 iptables 来执行。每一条 iptables 的规则都被 Linux 核心直接使用，被作为 kernel task 存在。

### 表

1. raw: 仅用于配置数据包，以使它们免于连接跟踪
2. filter: 默认表，通常是所有与防火墙相关行为发生的地方
3. nat: 被用来进行 NAT 的表
4. mangle: 用于特殊的数据包修改
5. security: 用来执行 MAC (mandatory access control) 规则

### 链

1. 在进行路由选择前处理数据包（PREROUTING）；
2. 处理流入的数据包（INPUT）；
3. 处理流出的数据包（OUTPUT）；
4. 处理转发的数据包（FORWARD）；
5. 在进行路由选择后处理数据包（POSTROUTING）。

### Rules

包过滤是基于 Rules（规则）的。Rules 由多个 matches （数据包必须满足的条件，能让数据包被过滤出来）和一个 target （对过滤出的数据包的行为）。

## iptables 包处理流程

iptables 工作流程图说明：

- 小写字母为表
- 大写字母为链
- 每一个从任何网卡进入的数据包，都要经过这样从上到下的过程来决定其最终的去向
- 有些数据包的最终目标就是 local process，有些数据包是从 local process 产生的，因此会与其他数据包的行为表现不一致。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230526222637.jpg)

[iptables Tutorial 1.2.2](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html#TRAVERSINGOFTABLES)

```bash
XXXXXXXXXXXXXXXXXX
                             XXX     Network    XXX
                               XXXXXXXXXXXXXXXXXX
                                       +
                                       |
                                       v
 +-------------+              +------------------+
 |table: filter| %3C---+        | table: nat       |
 |chain: INPUT |     |        | chain: PREROUTING|
 +-----+-------+     |        +--------+---------+
       |             |                 |
       v             |                 v
 [local process]     |           ****************          +--------------+
       |             +---------+ Routing decision +------%3E |table: filter |
       v                         ****************          |chain: FORWARD|
****************                                           +------+-------+
Routing decision                                                  |
****************                                                  |
       |                                                          |
       v                        ****************                  |
+-------------+       +------>  Routing decision  <---------------+
|table: nat   |       |         ****************
|chain: OUTPUT|       |               +
+-----+-------+       |               |
      |               |               v
      v               |      +-------------------+
+--------------+      |      | table: nat        |
|table: filter | +----+      | chain: POSTROUTING|
|chain: OUTPUT |             +--------+----------+
+--------------+                      |
                                      v
                               XXXXXXXXXXXXXXXXXX
                             XXX    Network     XXX
                               XXXXXXXXXXXXXXXXXX
```

## iptables Configuration

### 清空/重置 iptables

```bash
# 清空rule，并不会修改链的默认行为，ACCEPT 或DROP
iptables -F

# 删除用户定义链
iptables -t *table* -X *chain*
```

### 将 FORWARD 和 INPUT 默认规则改为 DROP

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
```

### 配置 INPUT 规则

```bash
# 向INPUT 链中添加规则
# 允许本地网段 192.168.3.0/24 使用SSH 访问本机
# -I, --insert chain [rulenum] rule-specification, 如果rulenum 是1，则规则添加到chain 的最头部
iptables -I INPUT -p tcp -s 192.168.3.0/24 --dport 22 -j ACCEPT

# 将INPUT 默认规则修改为DROP
# -P, --policy chain target, target 只有DROP 和ACCEPT
iptables -P INPUT DROP

# 测试icmp ping 包 - centos7_1
ping 192.168.3.160
PING 192.168.3.160 (192.168.3.160) 56(84) bytes of data.
^C
--- 192.168.3.160 ping statistics ---
6 packets transmitted, 0 received, 100% packet loss, time 5044ms

# 修改icmp 规则为REJECT - redhat 8
iptables -A INPUT -s 192.168.3.0/24 -p icmp -j REJECT

# 测试icmp ping 包 - centos7_1
[root@localhost yum.repos.d]# ping 192.168.3.160
PING 192.168.3.160 (192.168.3.160) 56(84) bytes of data.
From 192.168.3.160 icmp_seq=1 Destination Port Unreachable
From 192.168.3.160 icmp_seq=2 Destination Port Unreachable
^C
--- 192.168.3.160 ping statistics ---
2 packets transmitted, 0 received, +2 errors, 100% packet loss, time 1026ms

# 修改icmp 规则为ACCEPT - redhat 8
# -R, --replace chain rulenum rule-specification
iptables -R INPUT 2 -p icmp -s 192.168.3.0/24 -j ACCEPT

# 测试icmp ping 包 - centos7_1
[root@localhost ~]# ping 192.168.3.160
PING 192.168.3.160 (192.168.3.160) 56(84) bytes of data.
64 bytes from 192.168.3.160: icmp_seq=1 ttl=64 time=5.24 ms
64 bytes from 192.168.3.160: icmp_seq=2 ttl=64 time=52.0 ms
^C
--- 192.168.3.160 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 5.248/28.666/52.085/23.419 ms

# 修改icmp 规则，只过滤centos7_2 的地址192.168.3.162，放行来自192.168.3.0/24 的其他流量
iptables -R INPUT 2 -p icmp ! -s 192.168.3.162 -j ACCEPT
iptables -A INPUT -p icmp -s 192.168.3.162 -j REJECT -m comment --comment "192.168.3.162 is not welcome"

```

### 查看 iptables 规则

```bash
# 查看规则，并表明rules 顺序，方便添加修改rule

[root@ray ansible_project_1]# iptables -nvL --line-number
Chain INPUT (policy DROP 41242 packets, 72M bytes)
num   pkts bytes target     prot opt in     out     source               destination
1     1007 69972 ACCEPT     tcp  --  *      *       192.168.3.0/24       0.0.0.0/0            tcp dpt:22
2        3   252 ACCEPT     icmp --  *      *      !192.168.3.162        0.0.0.0/0
3        0     0 REJECT     icmp --  *      *       192.168.3.162        0.0.0.0/0            /* 192.168.3.162 is not welcome */ reject-with icmp-port-unreachable

Chain FORWARD (policy DROP 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 23912 packets, 2320K bytes)
num   pkts bytes target     prot opt in     out     source               destination

```

### 保存 iptables 规则

```bash
# 保存iptables 规则，否则下次重启rule 失效
service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables: [ OK ]
```

### Log

Target Log 可以被用来记录命中了 rule 的数据包。不像 ACCEPT 或 DROP target，数据包会在命中 rule 后继执行后面的 rule。这就意味着，如果我们想记录所有 DROP 的数据包，我们就需要在 drop 规则前创建一条和 drop rule 一样的规则并使用 -j LOG。但是这样降低了效率而且让事情变得复杂，我们可以通过创建一个 logdrop 链来替代这种方案。

```bash
# create droplog chain
iptables -N logdrop

# 创建droplog 规则
iptables -A logdrop -m limit --limit 5/m --limit-burst 10 -j LOG
iptables -A logdrop -j DROP

# 此时无论在任何地方想记录DROP 的数据包只要将-j 设置为logdrop 即可
iptables -A INPUT -p icmp -s 192.168.3.0/24 -j logdrop

# show verbose

[root@ray ~]# iptables -nvL --line-number
Chain INPUT (policy DROP 1340 packets, 1126K bytes)
num   pkts bytes target     prot opt in     out     source               destination
1      761 52712 ACCEPT     tcp  --  *      *       192.168.3.0/24       0.0.0.0/0            tcp dpt:22
2        0     0 logdrop    icmp --  *      *       192.168.3.0/24       0.0.0.0/0

Chain FORWARD (policy DROP 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 1301 packets, 171K bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain logdrop (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 LOG        all  --  *      *       0.0.0.0/0            0.0.0.0/0            limit: avg 5/min burst 10 LOG flags 0 level 4
2        0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0

```

#### Log Limit 解释

```bash
iptables -A logdrop -m limit --limit 5/m --limit-burst 10 -j LOG
# module: -m limit 使用limit 的module
# --limit 设置平均速率
# --limit-burst 设置初始速率(init burst rate)
```

#### 查看 Log

```bash
journalctl -k | grep "IN=.*OUT=.*" | less
Sep 12 16:40:42 ray.redhat8 kernel: IN=ens160 OUT= MAC=00:0c:29:cc:a6:10:a0:c5:89:a2:1a:ca:08:00 SRC=192.168.3.161 DST=192.168.3.160 LEN=84 TOS=0x00 PREC=0x00 TTL=64 ID=55272 DF PROTO=ICMP TYPE=8 CODE=0 ID=52485 SEQ=1
Sep 12 16:40:43 ray.redhat8 kernel: IN=ens160 OUT= MAC=00:0c:29:cc:a6:10:a0:c5:89:a2:1a:ca:08:00 SRC=192.168.3.161 DST=192.168.3.160 LEN=84 TOS=0x00 PREC=0x00 TTL=64 ID=55324 DF PROTO=ICMP TYPE=8 CODE=0 ID=52485 SEQ=2
Sep 12 16:40:44 ray.redhat8 kernel: IN=ens160 OUT= MAC=00:0c:29:cc:a6:10:a0:c5:89:a2:1a:ca:08:00 SRC=192.168.3.161 DST=192.168.3.160 LEN=84 TOS=0x00 PREC=0x00 TTL=64 ID=55892 DF PROTO=ICMP TYPE=8 CODE=0 ID=52485 SEQ=3
```

#### Block DoS

```bash
iptables -A INPUT -p tcp --dport 80 -m limit --limit 20/minute --limit-burst 100 -j ACCEPT
# adjust numerical value to meet your requirements
```

```bash
# protect more
echo 1 > /proc/sys/net/ipv4/ip_forward
echo 1 > /proc/sys/net/ipv4/tcp_syncookies
echo 0 > /proc/sys/net/ipv4/conf/all/accept_redirects
echo 0 > /proc/sys/net/ipv4/conf/all/accept_source_route
echo 1 > /proc/sys/net/ipv4/conf/all/rp_filter
echo 1 > /proc/sys/net/ipv4/conf/lo/rp_filter
echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout
echo 1800 > /proc/sys/net/ipv4/tcp_keepalive_time
echo 1 > /proc/sys/net/ipv4/tcp_window_scaling
echo 0 > /proc/sys/net/ipv4/tcp_sack
echo 1280 > /proc/sys/net/ipv4/tcp_max_syn_backlog
```

#### Block Port Scanning

```bash
# create a new chain block-scan
iptables -N block-scan
# create rule in it
iptables -A block-scan -p tcp -tcp-flags SYN,ACK,FIN,RST RST -m limit -limit 1/s -j RETURN
iptables -A block-scan -j DROP
```

#### Block Multiport Ports

```bash
badport='135,136,137,138,139,445'
iptables -A INPUT -p tcp -m multiport --dport $badport -j DROP
iptables -A INPUT -p udp -m multiport --dport $badport -j DROP
```
