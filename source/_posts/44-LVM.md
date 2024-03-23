---
title: 44.LVM
author: Rayd62
date: 2023-05-26 22:21:35
tags:
  - Linux
---

# LVM

## Background

LVM 解决了硬盘灵活性 (不容易调整分区大小，扩容困难) 的问题。

**LVM 有什么缺点吗？**

在如今计算、存储分离的环境下，LVM 还具备实用性吗？

在云计算环境中呢？

## Introduction

从根本上来讲，LVM 是将底层的物理卷 PV（磁盘）组成了一个大的资源池 VG 卷组。PE 是 LVM 的最小单元默认为 4M。当 Linux 需要使用到一块 block 来存储资料时，就可以简单的找 LVM 去按需申请一个逻辑卷。并且因为扩容方便，用户不需要在一开始就配置好未来很长时间才会使用的资源，这样运维人员可以按部就班的有计划的对存储设施进行升级。

![](https://cdn.jsdelivr.net/gh/Rayd62/note_images03@master/uPic/20230526185306.jpg)

### LVM 的扩容和缩小

LVM 的一个特点就是能够灵活的配置用户需要的存储。

扩容/缩小的异同点在于：扩容和缩容操作都需要先取消逻辑卷与目录的挂载关联；扩容操作是先扩容后检查文件系统完整性，而缩容操作为了保证数据的安全，需要先检查文件系统完整性再缩容。

另外在日常工作中，一定要谨慎使用或使用缩小的方案，因为这有数据丢失的风险。

### LVM 的快照功能

特点如下:

1. 快照卷 size = 逻辑卷 size。
2. 快照卷仅能使用一次，执行还原操作后会立即自动删除。

## Commands

| 功能/命令 | 物理卷管理 | 卷组管理  | 逻辑卷管理 |
| --------- | ---------- | --------- | ---------- |
| 扫描      | pvscan     | vgscan    | lvscan     |
| 创建      | pvcreate   | vgcreate  | lvcreate   |
| 显示      | pvdisplay  | vgdisplay | lvdisplay  |
| 删除      | pvremove   | vgremove  | lvremove   |
| 扩展      |            | vgextend  | lvextend   |
| 缩小      |            | vgreduce  | lvreduce   |

## Lab

### 部署

- 使硬盘支持 LVM 技术

```bash
# 使硬盘支持LVM 技术
[root@ray ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda             8:0    0    5G  0 disk
sdb             8:16   0    5G  0 disk
sr0            11:0    1  7.9G  0 rom
nvme0n1       259:0    0   20G  0 disk
|-nvme0n1p1   259:1    0  600M  0 part /boot/efi
|-nvme0n1p2   259:2    0    1G  0 part /boot
`-nvme0n1p3   259:3    0 18.4G  0 part
  |-rhel-root 253:0    0 16.4G  0 lvm  /
  `-rhel-swap 253:1    0    2G  0 lvm  [SWAP]
[root@ray ~]# pvcreate /dev/sd{a,b}
  Physical volume "/dev/sda" successfully created.
  Physical volume "/dev/sdb" successfully created.

```

- 创建 vg 卷组

```bash
[root@ray ~]# vgcreate my_storage /dev/sd{a,b}
  Volume group "my_storage" successfully created
[root@ray ~]# vgdisplay
###
--- Volume group ---
  VG Name               my_storage
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               9.99 GiB
  PE Size               4.00 MiB
  Total PE              2558
  Alloc PE / Size       0 / 0
  Free  PE / Size       2558 / 9.99 GiB
  VG UUID               q0tQPx-2J4f-ppTc-KmZu-qUEH-Hgqt-m5Dvql
```

- 创建 lv 卷

```bash
# 注意-L 指定150M 但是LV 的基础单位为4M，因此在创建后需为4 的倍数，最终获取152M 空间
# 也可以使用-l 来为LV 指定逻辑卷需要多少块基础PE，因此对下面的逻辑卷152M 即可写为-l 38 (152/4=38)
[root@ray ~]# lvcreate -n volume_1 -L 150M my_storage
  Rounding up size to full physical extent 152.00 MiB
  Logical volume "volume_1" created.
[root@ray ~]# lvdisplay
--- Logical volume ---
  LV Path                /dev/my_storage/volume_1
  LV Name                volume_1
  VG Name                my_storage
  LV UUID                Ce3q96-7Oc6-bjq7-lDaa-M5gd-0JFx-dU8yhV
  LV Write Access        read/write
  LV Creation host, time ray.redhat8, 2020-09-06 18:14:58 +0800
  LV Status              available
  # open                 0
  LV Size                152.00 MiB
  Current LE             38
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```

- 挂载 lv 卷

```bash
# 格式化volume_1 的lv 卷
[root@ray ~]# mkfs.xfs /dev/my_storage/volume_1
meta-data=/dev/my_storage/volume_1 isize=512    agcount=4, agsize=9728 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=38912, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
# 创建挂载点
[root@ray ~]# mkdir /lvm_storage
# 挂载
[root@ray ~]# mount -o defaults /dev/my_storage/volume_1 /lvm_storage/
# 查看
[root@ray ~]# df -hT
Filesystem                      Type      Size  Used Avail Use% Mounted on
devtmpfs                        devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs                           tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs                           tmpfs     2.0G  9.5M  2.0G   1% /run
tmpfs                           tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/rhel-root           xfs        17G  4.3G   13G  26% /
/dev/nvme0n1p2                  xfs      1014M  264M  751M  26% /boot
/dev/nvme0n1p1                  vfat      599M  6.9M  592M   2% /boot/efi
tmpfs                           tmpfs     392M  4.0K  392M   1% /run/user/0
/dev/mapper/my_storage-volume_1 xfs       147M  8.9M  138M   7% /lvm_storage
```

### 扩容

```bash
# 卸载卷
[root@ray ~]# umount /lvm_storage
# 扩容lv 卷
[root@ray ~]# lvextend -L 300M /dev/my_storage/volume_1
# 重新挂载
[root@ray ~]# mount -o defaults /dev/my_storage/volume_1 /lvm_storage/
# lvm 系统中逻辑卷大小已扩容
[root@ray ~]# lvdisplay
--- Logical volume ---
  LV Path                /dev/my_storage/volume_1
  LV Name                volume_1
  VG Name                my_storage
  LV UUID                Ce3q96-7Oc6-bjq7-lDaa-M5gd-0JFx-dU8yhV
  LV Write Access        read/write
  LV Creation host, time ray.redhat8, 2020-09-06 18:14:58 +0800
  LV Status              available
  # open                 1
  LV Size                300.00 MiB
  Current LE             75
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
# 但注意此时卷大小在系统中还是为152M
[root@ray ~]# df -hT
Filesystem                      Type      Size  Used Avail Use% Mounted on
devtmpfs                        devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs                           tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs                           tmpfs     2.0G  9.5M  2.0G   1% /run
tmpfs                           tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/rhel-root           xfs        17G  4.3G   13G  26% /
/dev/nvme0n1p2                  xfs      1014M  264M  751M  26% /boot
/dev/nvme0n1p1                  vfat      599M  6.9M  592M   2% /boot/efi
tmpfs                           tmpfs     392M  4.0K  392M   1% /run/user/0
/dev/mapper/my_storage-volume_1 xfs       147M  147M   20K 100% /lvm_storage
# 需要使用xfs_growfs 来使系统识别新扩容的存储
[root@ray ~]# xfs_growfs /dev/my_storage/volume_1
meta-data=/dev/mapper/my_storage-volume_1 isize=512    agcount=4, agsize=9728 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=38912, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 38912 to 76800
# 查看此时系统识别的卷大小
[root@ray ~]# df -hT
Filesystem                      Type      Size  Used Avail Use% Mounted on
devtmpfs                        devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs                           tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs                           tmpfs     2.0G  9.5M  2.0G   1% /run
tmpfs                           tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mapper/rhel-root           xfs        17G  4.3G   13G  26% /
/dev/nvme0n1p2                  xfs      1014M  264M  751M  26% /boot
/dev/nvme0n1p1                  vfat      599M  6.9M  592M   2% /boot/efi
tmpfs                           tmpfs     392M  4.0K  392M   1% /run/user/0
/dev/mapper/my_storage-volume_1 xfs       295M  148M  147M  51% /lvm_storage
```

### 快照

```bash
# 查看逻辑卷情况
[root@ray /]# vgdisplay
--- Volume group ---
VG Name               my_storage
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               9.99 GiB
  PE Size               4.00 MiB
  Total PE              2558
  Alloc PE / Size       75 / 300.00 MiB
  Free  PE / Size       2483 / %3C9.70 GiB
  VG UUID               q0tQPx-2J4f-ppTc-KmZu-qUEH-Hgqt-m5Dvql
# 创建逻辑卷快快照，注意大小需要与被创建快照的逻辑卷大小一致
[root@ray /]# lvcreate -L 300M -s -n SNAP /dev/my_storage/volume_1
# 查看快照卷
[root@ray /]# lvdisplay
--- Logical volume ---
  LV Path                /dev/my_storage/SNAP
  LV Name                SNAP
  VG Name                my_storage
  LV UUID                QCyMey-EIdW-Vh3i-hLd4-ezGn-bVG8-fXkoT2
  LV Write Access        read/write
  LV Creation host, time ray.redhat8, 2020-09-06 18:40:32 +0800
  LV snapshot status     active destination for volume_1
  LV Status              available
  # open                 0
  LV Size                300.00 MiB
  Current LE             75
  COW-table size         300.00 MiB
  COW-table LE           75
  Allocated to snapshot  0.00%
  Snapshot chunk size    4.00 KiB
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:5
```

- 测试恢复

```bash
# 在LVM 挂载点创建一些测试文件
[root@ray lvm_storage]# echo "test file 1" %3E 1.txt
[root@ray lvm_storage]# echo "test file 2" > 2.txt
[root@ray lvm_storage]# echo "test file 3" > 3.txt
[root@ray lvm_storage]# ll
total 12
-rw-r--r--. 1 root root 12 Sep  6 18:44 1.txt
-rw-r--r--. 1 root root 12 Sep  6 18:44 2.txt
-rw-r--r--. 1 root root 12 Sep  6 18:44 3.txt
# 创建snapshot 卷
[root@ray /]# lvcreate -L 300M -s -n snap /dev/my_storage/volume_1
  Logical volume "snap" created.
[root@ray /]# lvdisplay
--- Logical volume ---
  LV Path                /dev/my_storage/snap
  LV Name                snap
  VG Name                my_storage
  LV UUID                tTO0bI-HCGM-Sbc8-bV33-w1ve-kGBd-Y5MdjD
  LV Write Access        read/write
  LV Creation host, time ray.redhat8, 2020-09-06 18:45:35 +0800
  LV snapshot status     active destination for volume_1
  LV Status              available
  # open                 0
  LV Size                300.00 MiB
  Current LE             75
  COW-table size         300.00 MiB
  COW-table LE           75
  Allocated to snapshot  0.00%
  Snapshot chunk size    4.00 KiB
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:5
# 在snapshot 创建之后添加额外的文件
[root@ray lvm_storage]# dd if=/dev/zero of=tt count=1 bs=100M
# 开始恢复快照
[root@ray /]# umount /lvm_storage
[root@ray /]# lvconvert --merge /dev/my_storage/snap
  Merging of volume my_storage/snap started.
  my_storage/volume_1: Merged: 74.23%
  my_storage/volume_1: Merged: 100.00%
# 重新挂载，并查看挂载点内容是否与快照之前的状态一致
[root@ray /]# mount -a
[root@ray /]# cd /lvm_storage/
[root@ray lvm_storage]# ls
1.txt  2.txt  3.txt
```

### 删除

```bash
# 卸载挂载点
[root@ray /]# umount /lvm_storage
# 清除fstab 中相关挂载信息
[root@ray /]# vim /etc/fstab
# 删除lv 卷
[root@ray /]# lvremove /dev/my_storage/volume_1
Do you really want to remove active logical volume my_storage/volume_1? [y/n]: y
  Logical volume "volume_1" successfully removed
# 删除vg 卷组
[root@ray /]# vgremove my_storage
  Volume group "my_storage" successfully removed
# 删除pv 卷
[root@ray /]# pvremove /dev/sd{a,b}
  Labels on physical volume "/dev/sda" successfully wiped.
  Labels on physical volume "/dev/sdb" successfully wiped.
```

## 扩展

### 云环境下的磁盘扩容技术

[Amazon EBS 弹性卷](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ebs-modify-volume.html?icmpid=docs_ec2_console)
