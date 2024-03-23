---
title: 43.Raid 阵列卡
author: Rayd62
date: 2023-05-26 22:21:27
tags:
  - Linux
---

# Raid 阵列

## Backgroud

对于存储要求高可靠的同时又要求性能，并且成本不能太高

现在有相应的存储设备来做这样的事情，但是 raid 也是一个很好的解决方案

另外在云计算环境下，raid 也是一个非常好的选择，例如在 AWS 中，EBS（volume） 本身就具备高可用，这种环境下的 raid 0 的收益会非常高。

## 常见的 Raid 类型

### Raid 0

Riad 0 **读取、写入最快**：将数据写入 Raid 阵列中的不同磁盘，及对读取和写入的需求扩散到不同磁盘，这样读取、写入速度自然就快了（理论上 Raid 0 的读取、写入就是所有在阵列中的磁盘的 IOPS 总和），同样它对空间的浪费也是最小的，理论上阵列的大小及时所有磁盘的总和。但是缺点也很明显，**Raid 0 不支持校验，这样如果阵列中的磁盘受损是没办法恢复数据的**，但是在**云计算中本来就支持高可用的磁盘非常适合这种类型**。**最少两块磁盘**就可以组成 Raid 0 阵列了。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230526184511.jpg)

### Raid 1

Raid 1 读取、写入不变：将数据同时写入到多块磁盘中，这样**只要不是所有磁盘同时损坏，数据是不会丢失的**。**缺点是成本高**。**最少两块磁盘**就可以组成 Raid 1 阵列了。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230526184913.jpg)

[云计算 RAID 的六种应用场景](https://cloud.tencent.com/developer/article/1558911)

遗留问题：

1. 补充常见的 Raid 类型、其优缺点、要求
2. 实验
3. 操作方式
4. 云实验
5. Raid 为什么不需要分区表？

在 linux 2.6.28 版本之后，raid 就可以创建分区表了，可以通过—auto={mdp, part, p} 三个参数来创建

## 实验

### 配置 Raid 10

- 添加 4 块硬盘到虚拟机
- 查看硬盘添加情况

```bash
[root@ray_d62 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0   20G  0 disk
|-sda1          8:1    0  600M  0 part /boot/efi
|-sda2          8:2    0    1G  0 part /boot
`-sda3          8:3    0 18.4G  0 part
  |-rhel-root 253:0    0 16.4G  0 lvm  /
  `-rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0    5G  0 disk
sdc             8:32   0    5G  0 disk
sdd             8:48   0    5G  0 disk
sde             8:64   0    5G  0 disk
sr0            11:0    1 1024M  0 rom
```

- 创建 raid 10

```bash
# -n 指定多少块device 会加入到Raid -l level 设置raid 类型（等级） -C = --create -v = --verbose
[root@ray_d62 ~]# mdadm -Cv /dev/md0 -a yes -n 4 -l 10 /dev/sdb /dev/sdc /dev/sde /dev/sdd
mdadm: layout defaults to n2
mdadm: layout defaults to n2
mdadm: chunk size defaults to 512K
mdadm: size set to 5237760K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

## -C:create  -v:verbose -a: auto {yes,md,mdp,part,p}{NM}
## -n:specify the number of active devices in the array
## -l:set raid level.
```

- 查看 raid 设备

```bash
[root@ray_d62 ~]# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sat Sep  5 23:56:38 2020
        Raid Level : raid10
        Array Size : 10475520 (9.99 GiB 10.73 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Sun Sep  6 07:39:37 2020
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : ray_d62.linux_developer:0  (local to host ray_d62.linux_developer)
              UUID : adad1ba1:69db532e:8864e3ec:ab70ea3a
            Events : 22

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       64        2      active sync set-A   /dev/sde
       3       8       48        3      active sync set-B   /dev/sdd
```

### 卸载 Raid 设备

```bash
[root@ray_d62 ~]# umount /dev/md0
[root@ray_d62 ~]# mdadm -Ss
[root@ray_d62 ~]# mdadm --zero-superblock /dev/sd{b,c,e,d}
```

### 配置 Raid 0

- 创建 Raid 0

```bash
[root@ray_d62 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0   20G  0 disk
|-sda1          8:1    0  600M  0 part /boot/efi
|-sda2          8:2    0    1G  0 part /boot
`-sda3          8:3    0 18.4G  0 part
  |-rhel-root 253:0    0 16.4G  0 lvm  /
  `-rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0    5G  0 disk
sdc             8:32   0    5G  0 disk
sdd             8:48   0    5G  0 disk
sde             8:64   0    5G  0 disk
sr0            11:0    1 1024M  0 rom
[root@ray_d62 ~]# mdadm -Cv /dev/md0 -a yes -n 2 -l 0 /dev/sd{b,c}
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

- 查看 Raid 0

```bash
[root@ray_d62 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda             8:0    0   20G  0 disk
|-sda1          8:1    0  600M  0 part  /boot/efi
|-sda2          8:2    0    1G  0 part  /boot
`-sda3          8:3    0 18.4G  0 part
  |-rhel-root 253:0    0 16.4G  0 lvm   /
  `-rhel-swap 253:1    0    2G  0 lvm   [SWAP]
sdb             8:16   0    5G  0 disk
`-md0           9:0    0   10G  0 raid0
sdc             8:32   0    5G  0 disk
`-md0           9:0    0   10G  0 raid0
sdd             8:48   0    5G  0 disk
sde             8:64   0    5G  0 disk
sr0            11:0    1 1024M  0 rom
[root@ray_d62 ~]# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Sep  6 08:54:34 2020
        Raid Level : raid0
        Array Size : 10475520 (9.99 GiB 10.73 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun Sep  6 08:54:34 2020
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

            Layout : -unknown-
        Chunk Size : 512K

Consistency Policy : none

              Name : ray_d62.linux_developer:0  (local to host ray_d62.linux_developer)
              UUID : 747855fb:c5ee0899:d065dc45:6f6fc9b9
            Events : 0

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
```

- 挂载

```bash
[root@ray_d62 ~]# mkfs.xfs -f /dev/md0
log stripe unit (524288 bytes) is too large (maximum is 256KiB)
log stripe unit adjusted to 32KiB
meta-data=/dev/md0               isize=512    agcount=16, agsize=163712 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=2618880, imaxpct=25
         =                       sunit=128    swidth=256 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=8 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

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
/dev/md0              xfs        10G  105M  9.9G   2% /md
```

- 卸载

```bash
[root@ray_d62 ~]# umount /md
[root@ray_d62 ~]# mdadm -Ss
[root@ray_d62 ~]# ls /dev/md0
ls: cannot access '/dev/md0': No such file or directory
[root@ray_d62 ~]# ls /dev/ | grep md
[root@ray_d62 ~]# mdadm --zero-superblock /dev/sd{b,c}
```

### 配置备份盘

- 创建 raid

```bash
[root@ray_d62 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0   20G  0 disk
|-sda1          8:1    0  600M  0 part /boot/efi
|-sda2          8:2    0    1G  0 part /boot
`-sda3          8:3    0 18.4G  0 part
  |-rhel-root 253:0    0 16.4G  0 lvm  /
  `-rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0    5G  0 disk
sdc             8:32   0    5G  0 disk
sdd             8:48   0    5G  0 disk
sde             8:64   0    5G  0 disk
sr0            11:0    1 1024M  0 rom
# -x 指定spare-devices （备份盘）的数量
[root@ray_d62 ~]# mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sd{b,c,d} /dev/sde
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 5237760K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

- 查看 raid 信息

```bash
[root@ray_d62 ~]# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun Sep  6 09:12:36 2020
        Raid Level : raid5
        Array Size : 10475520 (9.99 GiB 10.73 GB)
     Used Dev Size : 5237760 (5.00 GiB 5.36 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Sun Sep  6 09:13:04 2020
             State : clean
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : ray_d62.linux_developer:0  (local to host ray_d62.linux_developer)
              UUID : 12fe5535:13e58f16:912f1fe5:14676839
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       4       8       48        2      active sync   /dev/sdd

       3       8       64        -      spare   /dev/sde
```

- 挂载

```bash
[root@ray_d62 ~]# mkfs.xfs -f /dev/md0
log stripe unit (524288 bytes) is too large (maximum is 256KiB)
log stripe unit adjusted to 32KiB
meta-data=/dev/md0               isize=512    agcount=16, agsize=163712 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=2618880, imaxpct=25
         =                       sunit=128    swidth=256 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, versionn=2
         =                       sectsz=512   sunit=8 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@ray_d62 ~]# mount -o defaults /dev/md0 /md
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
/dev/md0              xfs        10G  105M  9.9G   2% /md
```
