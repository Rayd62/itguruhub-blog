---
title: 39.Linux 存储结构和磁盘划分
author: Rayd62
date: 2023-05-26 22:06:22
tags:
  - Linux
---

# 存储结构和磁盘划分

## 目录结构

Linux 中一切都是文件，包括目录、字符设备、块设备、套接字等等。Linux 中的一切都从根“/” 目录开始，并按照文件系统层次化标准（FHS）采用树形结构来存放文件，如下图。Linux 中的**文件和目录名称是严格区分大小写的**。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230526164032.jpg)

Linux 系统中常见的目录名称及内容（注意结合实际情况，不要认死理）：

| 目录        | 文件内容                                                    |
| ----------- | ----------------------------------------------------------- |
| /boot       | 开机所需文件—内核、开机菜单以及所需配置文件等               |
| /dev        | 以文件形式存放任何设备与接口                                |
| /etc        | 配置文件                                                    |
| /home       | 用户主目录                                                  |
| /bin        | 存放单用户模式下还可以操作的命令（binary）                  |
| /lib        | 开机时用到的函数库，以及/bin 与/sbin 下面的命令要调用的函数 |
| /sbin       | 开机过程中需要的命令                                        |
| /media      | 用于挂载设备文件的目录                                      |
| /opt        | 放置第三方的软件                                            |
| /root       | 系统管理员的家目录                                          |
| /srv        | 一些网络服务的数据文件目录                                  |
| /tmp        | 任何人均可使用的“共享”临时目录                              |
| /proc       | 虚拟文件系统，例如系统内核、进程、外部设备及网络状态等      |
| /usr/local  | 用户自行安装的软件                                          |
| /usr/sbin   | Linux 系统开机时不会使用到的软件/命令/脚本                  |
| /usr/share  | 帮助与说明文件，也可放置共享文件                            |
| /var        | 主要存放经常变化的文件，如日志 (variable)                   |
| /lost+found | 当文件系统发生错误时，将一些丢失的文件片段存放在这里        |

## 绝对路径和相对路径

绝对路径：从根“/”目录开始的文件或目录位置

相对路径：从用户当前目录开始的文件或目录位置

## 物理设备命名规则

硬件设备也是文件，Linux 的系统内核中的 udev 设备管理器会自动的把硬件名称规范起来，目的是让用户通过设备名称对设备有一个大致的判断。

另外 udev 设备管理器的服务会一直以守护进程的形式运行并侦听内核发出的信息来管理/dev 下的设备文件。

常见硬件设备的文件名称：

| 硬件设备       | 文件名               |
| -------------- | -------------------- |
| IDE 设备       | /dev/hd[a-d]         |
| SCSI/SATA/U 盘 | /dev/sd[a-p]         |
| 软驱           | /dev/fd[0-1]         |
| 打印机         | /dev/lp[0-15]        |
| 光驱           | /dev/cdrom           |
| 鼠标           | /dev/mouse           |
| 磁带机         | /dev/st0 or /dev/ht0 |

## 磁盘分区

### 硬盘设备简介

硬盘设备是由大量扇区组成的，每个扇区的容量为 512 kb。每个硬盘的第一个扇区是最重要的，因为其中存储着主引导记录和分区表信息。

主引导记录 446 KB

分区信息： 4 x 16 KB

结束符： 2 KB

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230526165426.jpg)

因此，对于主分区来说，最多只能有 4 个分区，在一些时候，这样划分是不满足实际需求的。这时候就引入了扩展分区的概念。

扩展分区并不是实际的分区，而是将本来用作存储分区信息的主分区的 16KB 用来存储指向另一个分区的指针。

(待进一步了解补充，另一个分区是分区表还是什么？？？)

## Linux 文件系统

### 常用命令

```bash
# 分区
root@ray_d62 ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x32b86b54.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table

Command (m for help):
```

## 实验

### 挂载新硬盘

- 在虚拟机上挂在一块新的硬盘
- 查看硬盘在文件系统中的名称

```bash
[root@ray_d62 ~]# df -hT
Filesystem            Type      Size  Used Avail Use% Mounted on
devtmpfs              devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs                 tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs                 tmpfs     2.0G  9.4M  2.0G   1% /run
tmpfs                 tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/rhel-root xfs        17G  4.5G   12G  27% /
/dev/sda2             xfs      1014M  264M  751M  26% /boot
/dev/sda1             vfat      599M  6.9M  592M   2% /boot/efi
tmpfs                 tmpfs     392M  4.0K  392M   1% /run/user/0
[root@ray_d62 ~]# ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sda2  /dev/sda3  /dev/sdb
```

可以看出/dev/sda 是系统盘，已经挂在，而新出现的/dev/sdb 还未使用

- 给新硬盘分区

```bash
[root@ray_d62 ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xd54cf8f7.

Command (m for help): n # create new sector
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p   # primary disk
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-10485759, default 10485759): +5G
Value out of range.
Last sector, +sectors or +size{K,M,G,T,P} (2048-10485759, default 10485759):+5G
Value out of range. # can't assign all spaces to this disk(maybe caused by 2048 sector for Partition Table)
Last sector, +sectors or +size{K,M,G,T,P} (2048-10485759, default 10485759): 10485759

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): p # print the partition table
Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd54cf8f7

Device     Boot Start      End  Sectors Size Id Type
/dev/sdb1        2048 10485759 10483712   5G 83 Linux

Command (m for help): w # save change and quit
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

- 格式化磁盘

```bash
[root@ray_d62 ~]# mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=327616 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=1310464, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

- 挂载

```bash
[root@ray_d62 /]# mkdir mount_point
# 若/dev/sdb1 报错，可以使用命令刷新
[root@ray_d62 /]# partprobe
[root@ray_d62 /]# mount /dev/sdb1 /mount_point/
[root@ray_d62 /]# df -hT
Filesystem            Type      Size  Used Avail Use% Mounted on
devtmpfs              devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs                 tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs                 tmpfs     2.0G  9.4M  2.0G   1% /run
tmpfs                 tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/rhel-root xfs        17G  4.5G   12G  27% /
/dev/sda2             xfs      1014M  264M  751M  26% /boot
/dev/sda1             vfat      599M  6.9M  592M   2% /boot/efi
tmpfs                 tmpfs     392M  4.0K  392M   1% /run/user/0
/dev/sdb1             xfs       5.0G   68M  5.0G   2% /mount_point
```

- 添加 fstab 信息

```bash
# 在/etc/fstab 中添加下列信息
/dev/sdb1 /mount_point xfs defaults 0 0
```

- 重启查看是否开机自动挂载

```bash
[root@ray_d62 ~]# df -hT
Filesystem            Type      Size  Used Avail Use% Mounted on
devtmpfs              devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs                 tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs                 tmpfs     2.0G  9.4M  2.0G   1% /run
tmpfs                 tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/rhel-root xfs        17G  4.5G   12G  27% /
/dev/sda2             xfs      1014M  264M  751M  26% /boot
**/dev/sdb1             xfs       5.0G   68M  5.0G   2% /mount_point**
/dev/sda1             vfat      599M  6.9M  592M   2% /boot/efi
tmpfs                 tmpfs     392M  4.0K  392M   1% /run/user/0

# 自动挂载成功
```

### 创建新的 Swap 分区

- 在虚拟机中挂载一块 SCSI 硬盘
- 在系统中确认新硬盘的文件

```bash
[root@ray_d62 ~]# ls /dev/ | grep sd
sda
sda1
sda2
sda3
sdb
sdb1
sdc
[root@ray_d62 ~]# df -hT
Filesystem            Type      Size  Used Avail Use% Mounted on
devtmpfs              devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs                 tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs                 tmpfs     2.0G  9.4M  2.0G   1% /run
tmpfs                 tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/rhel-root xfs        17G  4.5G   12G  27% /
/dev/sdb1             xfs       5.0G   68M  5.0G   2% /mount_point
/dev/sda2             xfs      1014M  264M  751M  26% /boot
/dev/sda1             vfat      599M  6.9M  592M   2% /boot/efi
tmpfs                 tmpfs     392M  4.0K  392M   1% /run/user/0
```

确定新硬盘文件是/dev/sdc

- 新建分区表

```bash
[root@ray_d62 ~]# fdisk /dev/sdc

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xdaf75f95.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-2097151, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-2097151, default 2097151):

Created a new partition 1 of type 'Linux' and of size 1023 MiB.

Command (m for help): p
Disk /dev/sdc: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdaf75f95

Device     Boot Start     End Sectors  Size Id Type
/dev/sdc1        2048 2097151 2095104 1023M 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

- 格式化磁盘为 swap 分区

```bash
[root@ray_d62 ~]# mkswap /dev/sdc1
Setting up swapspace version 1, size = 1023 MiB (1072689152 bytes)
no label, UUID=3b5c866b-bd62-4365-b362-ffd189cc8f8e
```

- 将新分区添加到 swap 中

```bash
[root@ray_d62 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3912         332        3320           9         258        3344
Swap:          2047           0        2047
[root@ray_d62 ~]# swapon /dev/sdc1
[root@ray_d62 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3912         333        3310           9         267        3342
Swap:          3070           0        3070
```

- 将新的 swap 分区添加到/etc/fstab 中，便于下次启动自行加载

```bash
# 将下列配置添加到/etc/fstab 中
/dev/sdc1 swap swap defaults 0 0
```

- 重启检验

```bash
[root@ray_d62 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3912         340        3314           9         257        3336
Swap:          3070           0        3070
```
