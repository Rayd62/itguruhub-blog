---
title: 34.Mac OS 烧录ISO 镜像到U 盘 - 使用dd 命令
author: Rayd62
date: 2022-12-01 12:53:17
tags:
  - Linux
---

# 获取 U 盘路径

使用 `diskutil list` 查看系统已加载硬盘（U 盘），这里使用的是 `/dev/disk4`

```bash
> diskutil list
/dev/disk0 (internal):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                         251.0 GB   disk0
   1:             Apple_APFS_ISC ⁨⁩                        524.3 MB   disk0s1
   2:                 Apple_APFS ⁨Container disk3⁩         245.1 GB   disk0s2
   3:        Apple_APFS_Recovery ⁨⁩                        5.4 GB     disk0s3

/dev/disk3 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +245.1 GB   disk3
                                 Physical Store disk0s2
   1:                APFS Volume ⁨Macintosh HD - Data⁩     180.9 GB   disk3s1
   2:                APFS Volume ⁨Macintosh HD⁩            23.6 GB    disk3s3
   3:              APFS Snapshot ⁨com.apple.os.update-...⁩ 23.6 GB    disk3s3s1
   4:                APFS Volume ⁨Preboot⁩                 635.7 MB   disk3s4
   5:                APFS Volume ⁨Recovery⁩                1.6 GB     disk3s5
   6:                APFS Volume ⁨VM⁩                      1.1 GB     disk3s6

/dev/disk4 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *7.9 GB     disk4
   1:       Microsoft Basic Data ⁨⁩                        223.2 KB   disk4s1
   2:                        EFI ⁨NO NAME⁩                 2.9 MB     disk4s2
   3:                  Apple_HFS ⁨PVE⁩                     1.0 GB     disk4s3
   4:       Microsoft Basic Data ⁨⁩                        307.2 KB   disk4s4
```

# 卸载对应 U 盘

使用命令写在对应硬盘，按需要修改硬盘盘符 `diskutil unmountdisk /dev/disk4`

# 烧录

在 Mac（UNIX 系统）下可以使用 `dd` 命令来烧录，命令如下：

```bash
> sudo dd if=<iso> of=/dev/disk4 bs=4k
```

# 弹出 U 盘

使用命令 `diskutil eject /dev/disk4` 弹出对应 U 盘
