---
title: 35.FMC & FTD 安装
author: Rayd62
date: 2023-03-06 22:07:37
tags:
  - Cisco
  - Security
---

# 实验拓扑

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303061940497.png)

# 部署 FMC

> 实验使用的环境是 Esxi 6.7，没有安装 vCenter，因此无法在导入 OVF 文件时直接做配置，而是需要在进入到系统后，通过 CLI 来配置系统相关参数。

## 导入 OVF

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303061948867.png)

Esxi 环境使用红框中的配置文件 + VMDK 文件；

vCenter 环境使用蓝框中的配置文件 + VMDK 文件；

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303061949735.png)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303061950107.png)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303061951507.png)

## 初始化系统

系统导入完成后，打开电源开始系统安装，整个过程会持续 15 分钟以上，具体时间根据使用者的机器配置决定。

出现下面的信息就表示系统安装完毕，这时候敲回车就能正常看到登录提示符。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062016915.png)

默认用户名：admin 默认密码：Admin@123

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062017527.png)

登陆后，提示按“Enter”，阅读软件的最终用户许可协议。按提示输入“YES”，接受协议。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062018558.png)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062019243.png)

按要求设置密码。这里的密码是 Linux 的管理员密码 + web 登录密码。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062020867.png)

完成密码设置后，根据提示 FMC 管理口网络。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062023540.png)

确认配置无误后，输入“y“ 确认配置（注意根据实际情况做相应配置）。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062026749.png)

FMC 系统初始化完成。系统已经进入 CLI 模式，可以进行操作。

## 修改 FMC 管理口网络配置

使用命令 `expert` 进入 Linux 模式，使用命令 `sudo /usr/local/sf/bin/configure-network` 重新配置管理口网络。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314135851.jpg)

## 查看 FMC 接口地址

使用命令 `expert` 进入 Linux 模式，使用命令 `ifconfig` 就可以查看接口网络配置了。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062032270.png)

## 访问 FMC Web 界面

使用同网段的另一台主机访问 FMC 管理接口 172.16.1.200。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062049105.png)

# 部署 FTD

## 导入 OVF

与 FMC 一样，分别有 esxi 和 vcenter 两个版本的 ovf 配置文件，使用不同环境时需用对应文件 + vmdk 文件即可。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314135955.jpg)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140021.jpg)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140050.jpg)

配置接口到不同的端口组（在实际应用中应该是对应不同的服务器网卡）。红色框中的 4 个是必须配置的项，依次是管理口（Management 0/0）、Diagnostic 接口（Diagnostic）、Outside 接口（GigabitEthernet 0/0）和 Inside 接口（GigabitEthernet 0/1）。根据拓扑，我们实际还需配置 DMZ 和 VPN 两个接口，分别是 GigabitEthernet 0/2 、GigabitEthernet 0/3.

详细的接口划分如下表：

| Network Adapter    | Source Networks    | Destination Networks | Function                |
| ------------------ | ------------------ | -------------------- | ----------------------- |
| Network adapter 1  | Management0-0      | Management0/0        | Management              |
| Network adapter 2  | Diagnostic0-0      | Diagnostic0/0        | Diagnostic              |
| Network adapter 3  | GigabitEthernet0-0 | GigabitEthernet0/0   | Outside data            |
| Network adapter 4  | GigabitEthernet0-1 | GigabitEthernet0/1   | Inside date             |
| Network adapter 5  | GigabitEthernet0-2 | GigabitEthernet0/2   | Data traffic (Optional) |
| Network adapter 6  | GigabitEthernet0-3 | GigabitEthernet0/3   | Data traffic (Optional) |
| Network adapter 7  | GigabitEthernet0-4 | GigabitEthernet0/4   | Data traffic (Optional) |
| Network adapter 8  | GigabitEthernet0-5 | GigabitEthernet0/5   | Data traffic (Optional) |
| Network adapter 9  | GigabitEthernet0-6 | GigabitEthernet0/6   | Data traffic (Optional) |
| Network adapter 10 | GigabitEthernet0-7 | GigabitEthernet0/7   | Data traffic (Optional) |

[Cisco Secure Firewall Threat Defense Virtual Getting Started Guide, Version 7.3 - Deploy the Threat Defense Virtual on VMware [Cisco Secure Firewall Threat Defense Virtual]](https://www.cisco.com/c/en/us/td/docs/security/firepower/quick_start/consolidated_ftdv_gsg/threat-defense-virtual-73-gsg/m-ftdv-vmware-gsg.html)

部署类型，实验环境选择 4 Core / 8 GB。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140124.jpg)

点击“完成”，等待导入。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062042426.png)

## 初始化系统

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062050637.png)

看到登录提示界面，系统已安装完成。

使用默认账号：admin 默认密码：Admin123，登录系统。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140201.jpg)
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140237.jpg)

登陆后，提示按“Enter”，阅读软件的最终用户许可协议。按提示输入“YES”，接受协议。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140303.jpg)

按要求设置密码。这里的密码是防火墙的管理员密码。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140335.jpg)

根据提示配置 FTD 管理口网络。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062056893.png)

网络配置完成后提示本地管理 FTD 或使用 FMC 管理，填写”no” 即为使用 FMC 管理 FTD。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140415.jpg)

选择防火墙模式，这里选择默认的 `routed`。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140439.jpg)

完成初始化，进入 CLI 模式。

## 修改 FTD 管理口网络

在 FTD CLI 模式下，使用 `configure network ipv4 manual <mgmt0 IP> <netmask> <gateway> management0` 即可修改管理口地址。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062102926.png)

# FMC 获取测试许可

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062105070.png)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062104899.png)

# FTD 注册到 FMC

## FMC 配置默认放通所有流量的安全 Policy

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140509.jpg)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140533.jpg)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140553.jpg)

## FTD 配置注册

使用命令 `configure manager add <FMC-Mgmt IP> <register key>`。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062116610.png)

> 从 FMC 托管，使用命令 `configure manager delete`

## 在 FMC 中注册设备

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230314140619.jpg)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062119977.png)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062122895.png)

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03/202303062123069.png)

弹出提示，FMC 开始注册 FTD。
