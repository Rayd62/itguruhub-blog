---
title: 38.Linux 用户身份和文件权限
author: Rayd62
date: 2023-05-26 18:58:00
tags:
  - Linux
---

# 用户身份和文件权限

| 用户身份 | ID      | Remark                                                                     |
| -------- | ------- | -------------------------------------------------------------------------- |
| root     | 0       | 系统管理员，最大权限                                                       |
| 系统用户 | 1 - 999 | 避免服务出现漏洞导致黑客的提权攻击；默认服务程序会有对应的系统用户负责运行 |
| 普通用户 | ≥ 1000  | 由管理员创建；用户日常工作使用的账户                                       |

- UID 不可冲突；
- 即使前面有空闲的 UID，普通用户也要从 1000 开始

## 添加、修改和删除用户

### useradd - 添加用户

```bash

useradd [options] _username_

-d user home directory

-u assign a default UID

-g assign a basic group (group must exist)

-G assign one or more extend group

-s assign shell interpreter
```

### usermod - 修改用户属性

```bash

usermode [options] _username_

-c comment info

-d -m assign a new directory to user and migrate all data from old directory to new directory

-e set a account expire date YYYY-MM-DD

-g modify the default user group

-G modify the extend user group

-L lock the user, not allow login

-U unlock the user

-s change the default bash interpreter

-u change the UUID of the user
```

### userdel - 删除用户

```bash

userdel [option] username

-f delete user force

-r delete home directory of user at same time.
```

### groupadd

```bash

group [options] group_name
```

### passwd

```bash

passwd [options] username

普通用户只能修改自己的密码，且需要旧密码

root 用户可以修改所有人的密码，且无需旧密码

-l lock user

-u unlock user

—stdin allow change password by stdin input, echo "123" | passwd —stdin xingyuan 将xingyuan 的密码改为123

-e force user change password when login

-S display user status (lock status) and password (encrypted)
```

## 特殊权限

### SUID

只能对文件（或可执行文件）进行设置，可以让执行者（执行者必须拥有执行权限）临时拥有文件 owner 的权限。最直观的例子是/usr/bin/passwd

```bash
[cdadmin@SZ-szlicsta ~]$ ll /usr/bin/passwd
-rwsr-xr-x 1 root root 27856 Apr  1 11:57 /usr/bin/passwd

# 注意上面的s
```

passwd 的所有者属性为 rw**s**，与默认的 rwx 不一致，这就是 SUID 权限的标志，所有者属性的可执行权限变为 s。如果可执行文件的所有者没有执行权限 x，设置 SUID 后，其他用户的所有者属性为 S。注意分辨大小写

### SGID

SGID 的功能有：

1. 让执行者临时拥有所有者所属组的权限（对可执行文件设置）
2. 在某个目录中创建文件时自动继承该目录的用户组（对目录设置）

SGID 的第一种功能和 SUID 类似，只是将临时拥有文件 owner 的权限变为临时拥有文件所有组的权限。可以在/dev/ 中的块设备上看到这种设置。

例如：

```bash
[cdadmin@SZ-licwatcher ~]$ ll /dev/
total 0
crw-rw---- 1 root video    10, 175 Jul 14 16:26 agpgart
crw------- 1 root root     10, 235 Jul 14 16:26 autofs
--------------------------------------------------------
可以看到
```

SGID 的第二种功能在日常使用中会更多见。

# 目录权限总结

1. 目录拥有 rwx 权限

   拥有对目录的所有权限

   可以使用**ls**

   可以**创建、重命名（mv）、删除**所有在该**目录下的所有文件或目录**而不用关心这些文件或目录的**所有权或权限**

   可以进入目录

2. 目录拥有 rw 权限

   **不能**使用 ls - 可以查看目录，可以看到目录中的文件，但是看不到这些文件的所有者及权限

   不能进入目录（cd）

   不能执行当中的可执行文件

   不能新建、重命名（mv）、删除内容

   ```bash
   [xingyuan@SZ-licwatcher /]$ sudo chmod 600 /share/
   [xingyuan@SZ-licwatcher /]$ bash /share/test.sh
   bash: /share/test.sh: Permission denied
   [xingyuan@SZ-licwatcher /]$ ll /share/
   ls: cannot access /share/2: Permission denied
   ls: cannot access /share/1: Permission denied
   ls: cannot access /share/3: Permission denied
   ls: cannot access /share/test.sh: Permission denied
   total 0
   -????????? ? ? ? ?            ? 1
   -????????? ? ? ? ?            ? 2
   -????????? ? ? ? ?            ? 3
   -????????? ? ? ? ?            ? test.sh
   [xingyuan@SZ-licwatcher /]$ cd /share/
   -bash: cd: /share/: Permission denied
   [xingyuan@SZ-licwatcher /]$ ls /share/2
   ls: cannot access /share/2: Permission denied
   [xingyuan@SZ-licwatcher /]$ ll -d /share/
   drw------- 2 xingyuan xingyuan 33 Aug 11 14:17 /share/
   [xingyuan@SZ-licwatcher /]$ bash /share/test.sh
   bash: /share/test.sh: Permission denied
   [xingyuan@SZ-licwatcher /]$ mv /share/2 /share/2.bk
   mv: failed to access ‘/share/2.bk’: Permission denied
   [xingyuan@SZ-licwatcher /]$ rm -rf /share/2
   rm: cannot remove ‘/share/2’: Permission denied
   [xingyuan@SZ-licwatcher /]$ touch /share/4
   touch: cannot touch ‘/share/4’: Permission denied
   ```

3. 目录拥有 rx 权限

   可以使用 ls

   可以使用 cd

   可执行可执行文件

   不能新建、重命名（mv）、删除内容

   ```bash
   [xingyuan@SZ-licwatcher /]$ sudo chmod 500 /share/
   [xingyuan@SZ-licwatcher /]$ ll -d /share/
   dr-x------ 2 xingyuan xingyuan 48 Aug 11 14:22 /share/
   [xingyuan@SZ-licwatcher /]$ ll /share/
   total 4
   -rw-rw-r-- 1 cdadmin  cdadmin   0 Aug 11 14:17 1
   ---------- 1 xingyuan xingyuan  0 Aug 11 14:09 2
   -rw-rw-r-- 1 xingyuan xingyuan  0 Aug 11 14:17 3
   -rw-rw-r-- 1 xingyuan xingyuan 25 Aug 11 14:22 test.sh
   [xingyuan@SZ-licwatcher /]$ cd /share/
   [xingyuan@SZ-licwatcher share]$ cd ..
   [xingyuan@SZ-licwatcher /]$ bash /share/test.sh
   date
   [xingyuan@SZ-licwatcher /]$ touch /share/4
   touch: cannot touch ‘/share/4’: Permission denied
   [xingyuan@SZ-licwatcher /]$ mv /share/3 ~
   mv: cannot remove ‘/share/3’: Permission denied
   [xingyuan@SZ-licwatcher /]$ rm -rf /share/3
   rm: cannot remove ‘/share/3’: Permission denied
   ```
