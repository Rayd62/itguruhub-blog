---
title: 53.PXE 配置手册
author: Rayd62
date: 2023-05-30 11:07:39
tags:
  - Linux
  - Automation
---

# PXE 系统搭建手册

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230530110010.jpg)

PXE 装机过程：

1. 首先客户端广播请求 DHCP 服务器，数据包包含特定的 PXE 选项
2. DHCP 服务器回应相应的网络信息（分配给客户端的 IP 地址、网关等）给服务器，同时通过 bootstrap 协议（BOOTP）提供额外的信息（TFTP 地址，NBP(network bootstrap program) 初始下载路径，启动文件 (boot configuration file， pxelinux.cfg/default) 等）
3. PXE 服务器通过 TFTP 服务将 NBP （vmlinuz）转移到客户端，并将其加载到内存执行。Linux 核心和初始化文件通过这种方式完成转移。
4. 接着 root 文件系统 (initrd.img) 通过 HTTP/NFS/NBD 的任意一种方式完成传输。

# 配置详情

## DHCP Srv

```bash
yum install dhcp
{
allow booting;
allow bootp;                     #告知dhcp 服务器回应bootp 请求
ddns-update-style interim;
ignore client-updates;
subnet 192.168.62.0 netmask 255.255.255.0 {
        option subnet-mask      255.255.255.0;
        option domain-name-servers  192.168.62.10;
        range dynamic-bootp 192.168.62.100 192.168.62.200;
        default-lease-time      21600;
        max-lease-time          43200;
        next-server             192.168.62.10;        # 用于指定要从中加载初始引导文件（在filename 中指定文件）的服务器的主机地址。可以是IP或域名。此处为tftp ip
        filename                "RHEL76/pxelinux.0";  # 发行版bootloader file 第一个传输到客户端
}
```

> 注意，因为 filename 使用了一个目录为开头（因为此服务器提供多类 Linux 发行版，不同发行版不共用 pxelinux.0 文件），所以在之后 tftp 传输文件时，会有一个 tftp prefix 值为 RHEL76，意味着 pxe client 连接 tftp 的根目录为 RHEL76，而不是 tftp 配置文件中默认的/var/lib/tftpboot

`tcpdump -i ens33 host 192.168.62.10 and \!192.168.62.1 -w r1.pcap`  抓取流量查看 DHCP 过程，详细查看 option 的值

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230530110129.jpg)

## 配置 TFTP

```bash
yum install tftp-server
firewall-cmd --permanent --add-port=69/udp
firewall-cmd --reload

service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /var/lib/tftpboot   # tftp 服务的根目录
        disable                 = no                     # 修改默认值yes 为no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}
```

创建目录/var/lib/tftpboot/RHEL76 用于存放所有 RHEL7.6 版本安装所需文件

```bash
mkdir -p /var/lib/tftpboot/RHEL76
cd /var/lib/tftpboot/RHEL76

# 在PXE 装机时必要的4个文件分别是：pxelinux.0 pxelinux.cfg/default vmlinuz initrd.img
# 可以从挂载的光盘和syslinux 程序中获得
mount /dev/sr0 /media/cdrom
cp /media/cdrom/images/pxeboot/{vmlinuz,initrd.img} .
yum install syslinux -y
cp /usr/share/syslinux/pxelinux.0 .

# 可选拷贝文件，丢失不会影响安装，但是日志中会有报错
# 其中vesamenu.c32 是一个图形化的memu 系统，用来给管理员选择，安装pxe 服务器中的哪个系统；对应文字版的menu 系统为menu.c32
# 但是我们之后的pxelinux.cfg/default 配置文件中直接指定安装的镜像，跳过了在图形界面选操作系统安装
cp /media/cdrom/isolinux/{vesamenu.c32, boot.msg} .

[root@jyzq-yum RHEL76]# pwd
/var/lib/tftpboot/RHEL76
[root@jyzq-yum RHEL76]# ll
total 60188
-r--r--r--. 1 root root       84 Feb  9 14:14 boot.msg
-r--r--r--. 1 root root 54799220 Feb  9 14:14 initrd.img
-rw-r--r--. 1 root root    26759 Feb 18 11:22 pxelinux.0
drwxr-xr-x. 2 root root       34 Feb 23 12:11 pxelinux.cfg
-rw-rw-r--. 1 root root       61 Aug  3  2020 version.info
-r--r--r--. 1 root root   153104 Feb  9 14:14 vesamenu.c32
-r-xr-xr-x. 1 root root  6635920 Feb  9 14:14 vmlinuz
```

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230530110333.jpg)

pxe client 通过 tftp 服务器获取初始化所需文件和相应的配置文件

在 `pxelinux.cfg` 的目录中开始创建配置文件。

可以创建一个 `default`，该 `default` 可以为所有 PXE Client 提供。如果要对不同机器提供不同的配置，可以参考下列 URL 对配置进行命名。

常用的是针对 MAC 地址进行的命名配置，为 `01-aa-bb-cc-dd-ee-ff` (01 后面 +mac 地址，且为全部为小写)。

[PXELINUX](https://wiki.syslinux.org/wiki/index.php?title=PXELINUX)

可以从 RHEL 安装光盘，获取默认的 default 文件进行简单的修改就可以满足 PXE 的需求了。

`cp /media/cdrom/isolinux/isolinux.cfg pxelinux.cfg/default`

```
default linux
timeout 600

display boot.msg

# Clear the screen when exiting the menu, instead of leaving the menu displayed.
# For vesamenu, this means the graphical background is still displayed without
# the menu itself for as long as the screen remains in graphics mode.
menu clear
menu background splash.png
menu title Red Hat Enterprise Linux 7.6
menu vshift 8
menu rows 18
menu margin 8
#menu hidden
menu helpmsgrow 15
menu tabmsgrow 13

# Border Area
menu color border * #00000000 #00000000 none

# Selected item
menu color sel 0 #ffffffff #00000000 none

# Title bar
menu color title 0 #ff7ba3d0 #00000000 none

# Press [Tab] message
menu color tabmsg 0 #ff3a6496 #00000000 none

# Unselected menu item
menu color unsel 0 #84b8ffff #00000000 none

# Selected hotkey
menu color hotsel 0 #84b8ffff #00000000 none

# Unselected hotkey
menu color hotkey 0 #ffffffff #00000000 none

# Help text
menu color help 0 #ffffffff #00000000 none

# A scrollbar of some type? Not sure.
menu color scrollbar 0 #ffffffff #ff355594 none

# Timeout msg
menu color timeout 0 #ffffffff #00000000 none
menu color timeout_msg 0 #ffffffff #00000000 none

# Command prompt text
menu color cmdmark 0 #84b8ffff #00000000 none
menu color cmdline 0 #ffffffff #00000000 none

# Do not display the actual menu unless the user presses a key. All that is displayed is a timeout message.

menu tabmsg Press Tab for full configuration options on menu items.

menu separator # insert an empty line
menu separator # insert an empty line

#### 编辑这块配置

label linux
  menu label ^Install Red Hat Enterprise Linux 7.6
  kernel vmlinuz
  append initrd=initrd.img inst.repo=ftp://192.168.62.10/pub/RHEL76/ ks=ftp://192.168.62.10/pub/rhel76.cfg ip=dhcp quiet

#################

menu separator # insert an empty line

# utilities submenu
menu begin ^Troubleshooting
  menu title Troubleshooting

label vesa
  menu indent count 5
  menu label Install Red Hat Enterprise Linux 7.6 in ^basic graphics mode
  text help
        Try this option out if you're having trouble installing
        Red Hat Enterprise Linux 7.6.
  endtext
  ****kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=RHEL-7.6\\x20Server.x86_64 xdriver=vesa nomodeset quiet

label rescue
  menu indent count 5
  menu label ^Rescue a Red Hat Enterprise Linux system
  text help
        If the system will not boot, this lets you access files
        and edit config files to try to get it booting again.
  endtext
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=RHEL-7.6\\x20Server.x86_64 rescue quiet

label memtest
  menu label Run a ^memory test
  text help
        If your system is having issues, a problem with your
        system's memory may be the cause. Use this utility to
        see if the memory is working correctly.
  endtext
  kernel memtest

menu separator # insert an empty line

label local
  menu label Boot from ^local drive
  localboot 0xffff

menu separator # insert an empty line
menu separator # insert an empty line

label returntomain
menu label Return to ^main menu
  menu exit

menu end
```

在 PXE 的 label section 中有指明后续的 kickstart 的配置文件，`ks=ftp://192.168.62.10/pub/rhel76.cfg`

该文件描述了系统安装的各种配置相关，例如，分区、root 密码等

```bash
lang en_US.UTF-8 --addsupport=zh_CN.UTF-8,zh_HK.UTF-8,zh_TW.UTF-8
keyboard us
timezone Asia/Shanghai --isUtc
#rootpw $1$qW+xcSBY$.bCESCO5Rruf80Z2JgFcv0 --iscrypted
rootpw $1$8L2f59q3$JcTRnNGGVBwbu8I91xwKN1 --iscrypted
#platform x86, AMD64, or Intel EM64T
reboot
url --url=ftp://10.32.7.10/pub/RHEL76/
bootloader --location=mbr --append="rhgb quiet crashkernel=auto"
zerombr
clearpart --all --initlabel
part /boot --fstype="xfs" --ondisk=sda --size=1024
volgroup redhat --pesize=4096 pv.0
part pv.0 --fstype="lvmpv" --ondisk=sda --size=456320
logvol swap --vgname=redhat --name=swap --fstype=swap --size=32768
logvol / --vgname=redhat --name=os --fstype=xfs --size=423552
# part /mysql --fstype="xfs" --ondisk=sdb --size=xxxx
auth --usemd5 --useshadow
selinux --disabled
firewall --disable
firstboot --enable
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
%end
```

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230530110617.jpg)
