---
title: 49.网络配置 (nmcli&nmtui)
author: Rayd62
date: 2023-05-26 22:53:24
tags:
  - Linux
---

# 网络配置 (nmcli&nmtui)

## 常用命令

```bash
[root@SZ-licwatcher ~]# nmcli device
DEVICE       TYPE      STATE      CONNECTION
ens192       ethernet  connected  ens192
docker0      bridge    unmanaged  --
veth641d405  ethernet  unmanaged  --
lo           loopback  unmanaged  --

# -f option process output
[root@SZ-licwatcher ~]# nmcli -f DEVICE,STATE device
DEVICE       STATE
ens192       connected
docker0      unmanaged
veth641d405  unmanaged
lo           unmanaged

# -t option colon-separated format
[root@SZ-licwatcher ~]# nmcli -t  device
ens192:ethernet:connected:ens192
docker0:bridge:unmanaged:
veth641d405:ethernet:unmanaged:
lo:loopback:unmanaged:

# connection show
[root@SZ-licwatcher ~]# nmcli connection show
NAME    UUID                                  TYPE      DEVICE
ens192  03da7500-2101-c722-2438-d0d006c28c73  ethernet  ens192

# display the settings of a specific connection profile.
[root@SZ-licwatcher ~]# nmcli connection show ens192
connection.id:                          ens192
connection.uuid:                        03da7500-2101-c722-2438-d0d006c28c73
connection.stable-id:                   --
connection.type:                        802-3-ethernet
connection.interface-name:              ens192
connection.autoconnect:                 yes
connection.autoconnect-priority:        0
....

# modify properties of a connection
# multiple properties can be setted by using a single command
nmcli connection modify *connection_name property value [property value]*

# active a connection
nmcli connection up *connection_name*

# deactivate a connection
nmcli connection down *conection_name*

# check connection status:
[root@SZ-licwatcher ~]# nmcli networking connectivity
full
# none: host is not connected to any network.
# portal: the host is behind a captive portal and cannot reach the full Internet
# limited: the host is connected to a network, but it has no access to the Internet
# full: the host is connected to a network and has full access to the Internet
# unknown: the connectivity status cannot be found

[root@SZ-licwatcher ~]# nmcli connection
```

## 使用 Nmcli 配置静态以太网连接

### Purpose:

- static IPv4 - 192.0.2.1 with /24 subnet mask
- static IPv6 - 2001:db8:1::1 with /64 subnet mask
- IPv4 default gateway - 192.0.2.254
- IPv6 default gateway - 2001:db8:1::fffe
- IPv4 DNS server - 192.0.2.200
- An IPv6 DNS server - `2001:db8:1::ffbb`
- A DNS search domain - `example.com`

### Procedure:

- add new connection profile for the Ethernet connection:

```bash
# 注意ifname 需要与实际对应
[root@SZ-licwatcher ~]# nmcli connection add con-name test-connection ifname ens2020 type ethernet
Connection 'test-connection' (3a9d023a-9704-4421-b9db-f4a93ab95840) successfully added.
```

- set IPv4/IPv6 address:

```bash
[root@SZ-licwatcher ~]# nmcli connection modify *test-connection* ipv4.addresses *192.0.2.1/24*
[root@SZ-licwatcher ~]# nmcli connection modify *test-connection* ipv6.addresses *2001:db8:1::1/64*
```

- set IPv4 and IPv6 connection method to manual:

```bash
[root@SZ-licwatcher ~]# nmcli connection modify test-connection ipv4.method manual ipv6.method manual
```

- set IPv4 and IPv6 default gateway:

```bash
[root@SZ-licwatcher ~]# nmcli connection modify test-connection ipv4.gateway 192.0.2.254 ipv6.gateway 2001:db8:1::fffe
```

- set IPv4 and IPv6 DNS server:

```bash
[root@SZ-licwatcher ~]# nmcli con modify test-connection ipv4.dns 192.0.2.200 ipv6.dns 2001:db8:1::ffbb
```

- set IPv4 and IPv6 DNS search domain:

```bash
[root@SZ-licwatcher ~]# nmcli connection modify test-connection ipv4.dns-search example.com ipv6.dns-search example.com
```

- activate connection:

```bash
[root@SZ-licwatcher ~]# nmcli connection up test-connection
```

- Verify

```bash
# IP address verify
[root@SZ-licwatcher ~]# nmcli connection show test-connection | grep -E "ipv..address"
ipv4.addresses:                         192.0.2.1/24
ipv6.addresses:                         2001:db8:1::1/64

# Gateway (**ipv4 配置遇到bug，无法通过nmcli配置ipv4 的gateway**)
[root@SZ-licwatcher ~]# nmcli connection show test-connection | grep -E "ipv..gateway"
ipv4.gateway:                           --
ipv6.gateway:                           2001:db8:1::fffe

# 用另一种方式配置成功，如下：
[root@SZ-licwatcher ~]# nmcli connection modify test-connection +ipv4.gateway 192.0.2.254

# DNS
[root@SZ-licwatcher ~]# nmcli connection show test-connection | grep -E "*.dns"
connection.mdns:                        -1 (default)
ipv4.dns:                               192.0.2.200
ipv4.dns-search:                        example.com
ipv4.dns-options:                       ""
ipv4.dns-priority:                      0
ipv4.ignore-auto-dns:                   no
ipv6.dns:                               2001:db8:1::fff1
ipv6.dns-search:                        example.com
ipv6.dns-options:                       ""
ipv6.dns-priority:                      0
ipv6.ignore-auto-dns:                   no
```

- 配置 secondary IP

```bash
# 对配置ens160 添加第二IP 192.168.3.100/24
nmcli connection modify ens160 +ipv4.addresses 192.168.3.100/24

# 对配置ens160 删除第二IP 192.168.3.100/24
nmcli connection modify ens160 -ipv4.addresses 192.168.3.100/24
```

## Nmtui

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230526225257.jpg)
