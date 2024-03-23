---
title: 50.bonding&team
author: Rayd62
date: 2023-05-26 22:56:09
tags:
  - Linux
---

# bonding&team

## bonding 配置

通过 NetworkManager 守护进程来控制绑定接口。首先需要明确以下几点：

1. 启动主接口 (master interface) 不会自动启动从接口 (slave interface)
2. 启动从接口永远都会启动主接口
3. 停止主接口会自动停止从接口
4. 还没有从接口的主接口可以配置静态 IP
5. 当使用 DHCP 时，还没有从接口的主接口需要等待从接口的加入
6. A master with a DHCP connection waiting for slaves completes when a slave with a carrier is added.
7. A master with a DHCP connection waiting for slaves continues waiting when a slave without a carrier is added.

### Using Nmtui GUI to Configure a bonding Interface

[7.2. Configure Bonding Using the Text User Interface, nmtui Red Hat Enterprise Linux 7 | Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configure_bonding_using_the_text_user_interface_nmtui)

### Using Nmcli to Create Bond

#### Create Bond Connection

```bash
nmcli con add type bond ifname bond-name
# 因为没有对con-name 赋值，因此connection name 通过bond 的ifname 直接派生
```

NetworkManager Command Line (nmcli) 支持绝大多数 kernel 提供的 bonding 配置。例如：

```bash
nmcli con add type bond ifname bond-name bond.options "mode=balance-rr,miion=100"
```

#### Add a Slave Interface

添加 slave 网卡时，master connection 设置为 bond connection 名称。

```bash
nmcli con add type ethernet ifname ens33 master bond-name
```

激活从网卡

```bash
nmcli con up bond-slave-ens33
```

`active_slave`  和  `primary`  参数可以不影响 bond 接口工作的情况下进行修改。

```bash
nmcli dev mod bond-name +bond.options "active_slave=ens33"
nmcli dev mod bond-name +bond.options "primary=ens33"
```

### 使用 CLI 激活 bonding

在 RHEL 7 中，bonding 模块默认不打开。需要使用 root 权限加载模块：

```bash
modprobe --first-time bonding
# 该命令重启无效
```

## team 配置

```bash
nmcli connection add con-name team0 type team ifname team0
# 创建team0 的connection 类型team，创建新的设备team0

nmcli connection modify team0 team.runner loadbalance
# 修改team 模式为loadbalance

nmcli connection add con-name team0-port1 type team-slave ifname p5p1 master team0
nmcli connection add con-name team0-port2 type team-slave ifname p5p2 master team0
# 将p5p1 和p5p2 端口作为slave 加入team0
```
