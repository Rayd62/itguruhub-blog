---
title: 36.VMware 对虚拟机抓包
author: Rayd62
date: 2023-03-14 13:49:49
tags:
  - 运维
  - VMware
---

# 打开 ESXi 的 SSH

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/xWzGAd.png)

# 使用 SSH 登录 ESXi 后台

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230313133359.jpg)

# 获取虚拟机网卡编号

## 获取虚拟机网卡

记下需要抓包虚机的 `World ID`，这里以 `Site1_GW` 虚机为例，记下值 `2178744`

```bash
[root@vm1:~] esxcli network vm list
World ID  Name               Num Ports  Networks
--------  -----------------  ---------  --------------------------------------------------------------------------------------------------------------------------------------------
 2203241  Win_Nor                    2  Security_Lab_Site1_Inside, VM Network
 2099871  MediaSrv-Debian11          1  VM Network
 2119668  FMC_Lab                    1  VM Network
 2119833  FTD_Lab                    6  Security_Lab_Site1_VPN, Security_Lab_Site1_DMZ, Security_Lab_Site1_Inside, Security_Lab_Site1_Outside, Security_Lab_Site1_Inside, VM Network
 2121270  vyos_CPE_GW                3  Site2_to_PCE, Site1_to_CPE, VM Network
 2124597  Lab_ISE3                   2  Security_Lab_Site1_DMZ, VM Network
 2127622  Lab_Linux                  2  VM Network, Security_Lab_Site1_DMZ
 2148996  Lab_Kali                   2  Security_Lab_Site1_Outside, VM Network
 2173290  VCSA                       1  VM Network
 2178744  Site1_GW                   2  Security_Lab_Site1_Outside, Site1_to_CPE
 2199940  ros                        1  VM Network
[root@vm1:~]
```

## 获取网卡端口编号

```bash
[root@vm1:~] esxcli network vm port list -w 2178744
   Port ID: 33554578
   vSwitch: vSwitch0
   Portgroup: Security_Lab_Site1_Outside
   DVPort ID:
   MAC Address: 00:0c:29:47:03:dc
   IP Address: 0.0.0.0
   Team Uplink: vmnic0
   Uplink Port ID: 33554434
   Active Filters:

   Port ID: 33554579
   vSwitch: vSwitch0
   Portgroup: Site1_to_CPE
   DVPort ID:
   MAC Address: 00:0c:29:47:03:d2
   IP Address: 0.0.0.0
   Team Uplink: vmnic0
   Uplink Port ID: 33554434
   Active Filters:
```

可以看到 `Site1_GW` 的虚拟机下，有两张虚拟网卡，我们需要记下 `Port ID` 的值。这里记录 `33554578`。

# 使用 `pktcap-um` 抓包

`pktcap-uw --switchport 33554578 -c 2000 -o capture1.pcap`

这里 `--switchport` 后携带刚才记录的 `Port ID` ， `-c` 为可选项抓包数量， `-o` 为抓包结果输出的值。

> 详细抓包设置见
>
> [pktcap-uw 抓包工具](https://rayd62.notion.site/pktcap-uw-e25f6d3f5ec245ebb35d9994a61ebe82)

之后使用 `scp` 导出文件，即可使用 `wireshark` 等包分析工具。
