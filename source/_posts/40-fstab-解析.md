---
title: 40.fstab 解析
author: Rayd62
date: 2023-05-26 22:06:37
tags:
  - Linux
---

# fstab 解析

fstab(5) 文件可用于定义磁盘分区，各种其他块设备或远程文件系统应如何装入文件系统。

在/etc/fstab 中描述的磁盘分区会在系统启动时自行加载。

## Example

```bash
[root@ray_d62 ~]# cat /etc/fstab  | grep -v "^#"
# <file system>        <dir>         <type>    <options>             <dump> <pass>
/dev/mapper/rhel-root   /    xfs      defaults        0 0
UUID=e52aa668-070e-4ed9-b025-123b8a7d731b /boot  xfs     defaults        0 0
UUID=248F-3EF4          /boot/efi     vfat    umask=0077,shortname=winnt 0 2
/dev/mapper/rhel-swap   swap          swap    defaults        0 0
/dev/sdb1               /mount_point  xfs defaults,uquota 0 0
/dev/sdc1 swap swap defaults 0 0
```

## 字段定义

该文件包含以下字段，并用 "tab" 分割：

```bash
<file system>	<dir>	<type>	<options>	<dump>	<pass>
```

- \<file system\>: 要挂载的分区或存储设备
- \<dir\>: \<file systems\>的挂载位置。
- \<type\> - 要挂载设备或是分区的文件系统类型，支持许多种不同的文件系统：ext2, ext3, ext4, reiserfs, xfs, jfs, smbfs, iso9660, vfat, ntfs, swap  及  auto。 设置成 auto 类型，mount 命令会猜测使用的文件系统类型，对 CDROM 和 DVD 等移动设备是非常有用的。
- \<option\>: 挂载时使用的参数，注意有些 参数是特定文件系统才有的。一些比较常用的参数有 ([mount(8)](https://jlk.fjfi.cvut.cz/arch/manpages/man/mount.8))：
  - `auto`：在启动时载入
  - `mount -a ` 命令时自动挂载。
  - `noauto`：只在你的命令下被挂载。
  - `exec`：允许执行此分区的二进制文件。
  - `noexec`：不允许执行此文件系统上的二进制文件。
  - `ro`：以只读模式挂载文件系统。
  - `rw`：以读写模式挂载文件系统。
  - `user`：允许任意用户挂载此文件系统，若无显示定义，隐含启用  `noexec`, `nosuid`, `nodev`  参数。
  - `users`：允许所有 users 组中的用户挂载文件系统.
  - `nouser`：只能被 root 挂载。
  - `owner`：允许设备所有者挂载.
  - `sync`：I/O 同步进行。
  - `async`：I/O 异步进行。
  - `dev`：解析文件系统上的块特殊设备。
  - `nodev`：不解析文件系统上的块特殊设备。
  - `suid`：允许 suid 操作和设定 sgid 位。这一参数通常用于一些特殊任务，使一般用户运行程序时临时提升权限。
  - `nosuid`：禁止 suid 操作和设定 sgid 位。
  - `noatime`：不更新文件系统上 inode 访问记录，可以提升性能 (参见  [atime 参数](<https://wiki.archlinux.org/index.php/Fstab_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#atime_%E5%8F%82%E6%95%B0>))。
  - `nodiratime`：不更新文件系统上的目录 inode 访问记录，可以提升性能 (参见  [atime 参数](<https://wiki.archlinux.org/index.php/Fstab_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#atime_%E5%8F%82%E6%95%B0>))。
  - `relatime`：实时更新 inode access 记录。只有在记录中的访问时间早于当前访问才会被更新。（与 noatime 相似，但不会打断如 mutt 或其它程序探测文件在上次访问后是否被修改的进程。），可以提升性能 (参见  [atime 参数](<https://wiki.archlinux.org/index.php/Fstab_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#atime_%E5%8F%82%E6%95%B0>))。
  - `flush` - `vfat`：更频繁的刷新数据，复制对话框或进度条在全部数据都写入后才消失。
  - `defaults`：使用文件系统的默认挂载参数，例如  `ext4`  的默认参数为: `rw`, `suid`, `dev`, `exec`, `auto`, `nouser`, `async`.
- \<pass\>：fsck 读取 \<pass\> 的数值来决定需要检查的文件系统的检查顺序。允许的数字是 0, 1, 和 2。  根目录应当获得最高的优先权 1,  其它所有需要被检查的设备设置为  2. 0  表示设备不会被 fsck  所检查。
- \<dump\>：dump 工具通过它决定何时作备份. dump 会检查其内容，并用数字来决定是否对这个文件系统进行备份。 允许的数字是 0 和 1 。0 表示忽略， 1 则进行备份。大部分的用户是没有安装 dump 的 ，对他们而言 \<dump\> 应设为 0。

# Resources

[fstab](https://wiki.archlinux.org/index.php/Fstab_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
