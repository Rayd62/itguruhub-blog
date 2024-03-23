---
title: 46.firewalld
author: Rayd62
date: 2023-05-26 22:35:30
tags:
  - Linux
  - 安全
---

# firewalld

## 什么是 firewalld

Firewalld 提供了动态管理的防火墙，该防火墙支持网络/防火墙区域，该区域定义了网络连接或接口的信任级别。

它运行时和永久的配置分开。（有临时规则和永久规则）

它还为应用程序和服务提供了一个接口来直接添加防火墙规则。

它的前任防火墙模型是一个具有 system-config-firewall/lokkit 的静态防火墙且没次修改都需要完全重启的防火墙

> 以前带有 system-config-firewall / lokkit 的防火墙模型是静态的，每次更改都需要重新启动防火墙。  
> 这还包括卸载防火墙 netfilter 内核模块和加载新配置所需的模块。  
> 模块的卸载破坏了状态防火墙和已建立的连接。  
> 另一方面，防火墙守护程序动态管理防火墙并应用更改，而无需重新启动整个防火墙。  
> 因此，无需重新加载所有防火墙内核模块。  
> 但是，使用防火墙后台驻留程序要求所有对该防火墙后台驻留程序的修改都必须确保该后台驻留程序中的状态与内核中的防火墙处于同步状态。  
> 防火墙守护程序无法解析 iptables 和 ebtables 命令行工具添加的防火墙规则。  
> 该守护程序通过 D-BUS 提供有关当前活动防火墙设置的信息，并使用 PolicyKit 身份验证方法通过 D-BUS 接受更改。 —https://www.unixmen.com/iptables-vs-firewalld/

**注意：即使使用 reload 命令重载了 firewalld，已经建立的 session 和 connection 不会收到新的 rule 的影响。**

所以 firewalld 使用区域和服务而不是链和规则来执行操作，并且它允许通过动态更新&修改而不用破坏现有的回话和连接。

### 它具有以下特点：

- D-Bus API
- 定时的防火墙规则
- 用于特定防火墙规则的丰富语句 (rich language for specifi firewall rules)
- 支持 IPv4 和 IPv6 NAT
- Firewall Zone
- 支持 IP 集（IP set support）
- 对拒绝数据包的简单日志
- 直接接口 (Direct interface)
- 锁定：将可能修改防火墙的应用程序列入白名单。(Lockdown: whitelisting of applications that may modify the firewall)
- 支持 iptables, ip6tables, ebtables 和 ipset firewall backends
- 自动加载 Linux 内核模块
- 与 Puppet 集成

## 基础命令

| 参数                            | 功能                                                 |
| ------------------------------- | ---------------------------------------------------- |
| --get-default-zone              | 查询默认的区域名称                                   |
| --set-default-zone=\<区域名称\> | 设置默认的区域，使其永久生效                         |
| --get-zones                     | 显示可用的区域                                       |
| --get-services                  | 显示预先定义的服务                                   |
| --get-active-zones              | 显示当前正在使用的区域与网卡名称                     |
| --add-source=                   | 将源自此 IP 或子网的流量导向指定的区域               |
| --remove-source=                | 不再将源自此 IP 或子网的流量导向某个指定区域         |
| --add-interface=\<网卡名称\>    | 将源自该网卡的所有流量都导向某个指定区域             |
| --change-interface=\<网卡名称\> | 将某个网卡与区域进行关联                             |
| --list-all                      | 显示当前区域的网卡配置参数、资源、端口以及服务等信息 |
| --list-all-zones                | 显示所有区域的网卡配置参数、资源、端口以及服务等信息 |
| --add-service=\<服务名称\>      | 设置默认区域允许该服务的流量                         |
| --add-port=\<端口号/协议\>      | 设置默认区域允许该端口的流量                         |
| --remove-service=\<服务名称\>   | 设置默认区域不再允许该服务的流量                     |
| --remove-port=\<端口号/协议\>   | 设置默认区域不再允许该端口的流量                     |
| --reload                        | 让“永久生效”的配置规则立即生效，并覆盖当前的配置规则 |
| --panic-on                      | 开启应急状况模式                                     |
| --panic-off                     | 关闭应急状况模式                                     |

### 查看 firewalld 服务状态

```bash
[root@ray ~]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor prese%3E
   Active: active (running) since Sun 2020-09-13 11:29:15 CST; 4h 8min ago
     Docs: man:firewalld(1)
 Main PID: 1101 (firewalld)
    Tasks: 2 (limit: 24536)
   Memory: 32.8M
   CGroup: /system.slice/firewalld.service
           └─1101 /usr/libexec/platform-python -s /usr/sbin/firewalld --nofork --no>

Sep 13 11:29:14 ray.redhat8 systemd[1]: Starting firewalld - dynamic firewall daemo>
Sep 13 11:29:15 ray.redhat8 systemd[1]: Started firewalld - dynamic firewall daemon.
Sep 13 11:29:16 ray.redhat8 firewalld[1101]: WARNING: AllowZoneDrifting is enabled.>
lines 1-13/13 (END)
```

### 查看当前 Zone

#### 打印当前活动区域，以及区域中使用的接口和源

```bash
# --get-active-zones
#           Print currently active zones altogether with interfaces and sources
#           used in these zones. Active zones are zones, that have a binding to an
#           interface or source. The output format is:
#               zone1
#                 interfaces: interface1 interface2 ..
#                 sources: source1 ..
#               zone2
#                 interfaces: interface3 ..
#               zone3
#                 sources: source2 ..
[root@ray ~]# firewall-cmd --get-active-zones
libvirt
  interfaces: virbr0
public
  interfaces: ens160

```

#### 打印预定义的 Zones

```bash
# 打印预定义的zones 并用空格分割
[root@ray ~]# firewall-cmd --get-zones
block dmz drop external home internal libvirt public trusted work
```

#### 打印接口和连接的默认区域

```bash
[root@ray ~]# firewall-cmd --get-default-zone
public
```

#### 打印预先定义的 Services

```bash
# 以空格分割
[root@ray ~]# firewall-cmd --get-services
RH-Satellite-6 amanda-client amanda-k5-client amqp amqps apcupsd audit bacula bacula-client bb bgp bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc bittorrent-lsd ceph ceph-mon cfengine cockpit condor-collector ctdb dhcp dhcpv6 dhcpv6-client distcc dns dns-over-tls docker-registry docker-swarm dropbox-lansync elasticsearch etcd-client etcd-server finger freeipa-4 freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp ganglia-client ganglia-master git grafana gre high-availability http https imap imaps ipp ipp-client ipsec irc ircs iscsi-target isns jenkins kadmin kdeconnect kerberos kibana klogin kpasswd kprop kshell kube-apiserver ldap ldaps libvirt libvirt-tls lightning-network llmnr managesieve matrix mdns memcache minidlna mongodb mosh mountd mqtt mqtt-tls ms-wbt mssql murmur mysql nfs nfs3 nmea-0183 nrpe ntp nut openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole plex pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy prometheus proxy-dhcp ptp pulseaudio puppetmaster quassel radius rdp redis redis-sentinel rpc-bind rsh rsyncd rtsp salt-master samba samba-client samba-dc sane sip sips slp smtp smtp-submission smtps snmp snmptrap spideroak-lansync spotify-sync squid ssdp ssh steam-streaming svdrp svn syncthing syncthing-gui synergy syslog syslog-tls telnet tentacle tftp tftp-client tile38 tinc tor-socks transmission-client upnp-client vdsm vnc-server wbem-http wbem-https wsman wsmans xdmcp xmpp-bosh xmpp-client xmpp-local xmpp-server zabbix-agent zabbix-server
```

#### 查看某一接口所在的区域

```bash
[root@ray ~]# firewall-cmd --get-zone-of-interface=ens160
public
```

### 修改策略

#### 将某一个接口划分到另一个区域

```bash
# 临时划分
[root@ray ~]# firewall-cmd --zone=home --change-interface=ens160
success

# 永久划分，重启生效
[root@ray ~]# firewall-cmd --zone=home --change-interface=ens160 --permanent
The interface is under control of NetworkManager, setting zone to 'home'.
success
```

#### 修改默认区域为 Home

```bash
[root@ray ~]# firewall-cmd --set-default-zone=home
success
[root@ray ~]# firewall-cmd --get-default-zone
home
```

#### 启动/关闭紧急模式

```bash
# 断开一切网络连接
[root@ray ~]# firewall-cmd --panic-on
# 恢复网络连接
[root@ray ~]# firewall-cmd --panic-off
```

#### 查看区域是否允许 SSH 和 HTTPS 协议的流量

```bash
# 通过服务查询
[root@ray ~]# firewall-cmd --zone=public --query-service=ssh
yes
[root@ray ~]# firewall-cmd --zone=public --query-service=https
no
```

#### 允许 HTTPS 流量进入 Public 区域，并立即生效

```bash
[root@ray ~]# firewall-cmd --zone=public --add-service=https
success
[root@ray ~]# firewall-cmd --zone=public --add-service=https --permanent
success
[root@ray ~]# firewall-cmd --reload
success
```

#### 在 Home 区域中永久拒绝 Https 的流量

```bash
[root@ray ~]# firewall-cmd --zone=home --remove-service=https --permanent
Warning: NOT_ENABLED: https
success
[root@ray ~]# firewall-cmd --reload
success
```

#### 临时允许访问 Public 的 8080 和 8081 端口

```bash
[root@ray ~]# firewall-cmd --zone=public --add-port=8080-8081/tcp
success
[root@ray ~]# firewall-cmd --zone=public --list-ports
123/udp 8080-8081/tcp
[root@ray ~]#
```

#### 将访问本机的 888 端口流量转发到 22 端口

```bash
[root@ray ~]# firewall-cmd --permanent --zone=home --add-forward-port=port=888:proto=tcp:toport=22:toaddr=192.168.3.160
success
[root@ray ~]# firewall-cmd --reload
success
[root@ray ~]# firewall-cmd --zone=home --list-all
home (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports: port=888:proto=tcp:toport=22:toaddr=192.168.3.160
  source-ports:
  icmp-blocks:
  rich rules:
```

#### 拒绝来自 192.168.3.161 的 SSH 流量

```bash
[root@ray ~]# firewall-cmd --zone=home --add-rich-rule='rule
> family="ipv4"
> source address="192.168.3.161/32"
> service name="ssh"
> reject'
success

## 该命令及时生效不需要reload，注意是否需要变成--permanent
## 同时需要注意，如果192.168.3.161 已经建立了SSH 的session 到本机，那么该命令是无法干掉这个会话的
```
