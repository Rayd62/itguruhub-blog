---
title: 54.PXE 使用手册
author: Rayd62
date: 2023-05-30 11:12:39
tags:
  - Linux
  - Automation
---

# PXE 系统使用手册

使用 iDRAC 登录 Linux PXE 服务器.（DELL 服务器）

## 服务器信息

| IP           | 账号 | 密码  |
| ------------ | ---- | ----- |
| 10.32.92.241 | root | xxxxx |

## 修改 DHCP 配置文件

根据需要安装的操作系统版本的 pxelinux.0 所在位置更新  `/etc/dhcp/dhcpd.conf`  配置文件 。

推荐在 tftp 的默认目录/var/lib/tftpboot 根据不同操作系统创建不同目录。在 tftpd.conf 文件中将 pxelinux.0 的目录改为  `directory/pxelinux.0`

```bash
...
filename   "RHEL76/pxelinux.0";      # 即/var/lib/tftpboot/RHEL76 目录下的pxelinux.0 文件位置
...
```

## 准备必要文件

将对应操作系统版本的 `boot.msg`, `initrd.img`, `pxelinux.0`, `version.info`, `vesamenu.c32`, `vmlinuz` 文件拷贝到新建好的目录。（不同操作系统、不同版本可能所需文件不同，可以提前搜索好）

在目录下创建**目录**`pxelinux.cfg`

在 `pxelinux.cfg` 目录中，需要准备 PXE 第一阶段的配置文件。

需要修改的有两部分。

1. 配置文件的命名，其命名规则为 "01-xx-xx-xx-xx-xx-xx"，xx 使用服务器 PXE 网卡的 mac 地址替代，全部为小写字母；若所有服务器使用同一 kickstart 配置，则可以命名为 default
2. 配置文件修改

```bash
label linux
  menu label ^Install Red Hat Enterprise Linux 7.6
  kernel vmlinuz
  append initrd=initrd.img inst.repo=ftp://192.168.62.10/pub/RHEL76/ ks=ftp://192.168.62.10/pub/rhel76.cfg  quiet
```

```
 注意修改ftp 为项目实际的FTP 目录，ks 为项目所用的kickstart 文件。
```

### 准备 Kickstart 文件

`kickstart` 文件描述了操作系统安装后的状态，`bootloader` 读取到该信息，即会使用这些信息来安装操作系统。

kickstart 文件位置需放在上稳 `pxelinux.cfg` 目录中配置文件指定的位置。

在本环境里为 `ftp://192.168.62.10/pub/rhel76.cfg`，在 ftp 服务器上的实际地址为 `/var/ftp/pub/rhel76.cfg`

```bash
# 语言包，根据项目修改
lang en_US --addsupport=zh_CN.utf-8,zh_HK.UTF-8,zh_TW.UTF-8

# 键盘类型，无需修改
keyboard us

# 时区信息，无需修改
timezone Asia/Shanghai --isUtc

# root 密码加密后的字符串，默认密码'1234/.,m';lk'
rootpw $1$8L2f59q3$JcTRnNGGVBwbu8I91xwKN1 --iscrypted

#platform x86, AMD64, or Intel EM64T
# 安装后是否重启
reboot

# 安装文件的源地址，根据项目调整
url --url=ftp://192.168.62.10/pub/RHEL76/
# bootloader 选项，无需修改
bootloader --location=mbr --append="rhgb quiet crashkernel=auto"
zerombr
clearpart --all --initlabel

# 磁盘分区，大小为MB，其中--grow 属性意为使用剩余所有空间
part /boot --fstype="xfs" --ondisk=sda --size=1024
volgroup rhel --pesize=4096 pv.0
part pv.0 --fstype="lvmpv" --ondisk=sda --size=18000
logvol swap --vgname=rhel --name=swap --fstype=swap --size=2048
logvol / --vgname=rhel --name=os --fstype=xfs --grow --size=15000

# 密码加密选项，无需修改
auth --usemd5 --useshadow
# selinux 配置，根据项目修改
selinux --disabled
# 防火墙配置，可以disable，根据项目修改
firewall --enabled --ssh
firstboot --enable

# 安装系统时，安装软件包，根据项目修改
%packages
@^minimal
@compat-libraries
@core
@development
kexec-tools
apr
apr-util
subversion
telnet
vim
libpcap
lsof
net-tools
%end
```
