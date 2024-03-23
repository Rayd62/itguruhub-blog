---
title: 52.sudo
author: Rayd62
date: 2023-05-27 23:55:44
tags:
  - Linux
---

# Sudo

## 什么是 Sudo

因为 Linux 是一个多用户多任务的操作系统，一个系统往往不止一个用户使用。这样出于安全性考虑，就不能让所有用户都对系统拥有完整的控制权限（管理员权限/root），但是有一些操作用户必须使用 root 才能执行，在必要时让某些用户提权到 root，执行一些操作。这就是 sudo 设计出来的目的。

### 基本用法

```bash
# 提权到ray 用户权限，查看ray 的家目录内容
sudo -u ray ls -al /home/ray
```

注意当使用 sudo 提权后，所有的操作都是以提权后的身份执行。

例如当用户 A 使用 sudo -u user_b touch /home/user_b/1.tmp，此时文件 1.tmp 的所有者是 user_b

```bash
[xingyuan@SZ-licwatcher ~]$ sudo -u cdadmin vim /home/cdadmin/1.tmp
[cdadmin@SZ-licwatcher ~]$ ll
--------
-rw-r--r--  1 cdadmin cdadmin          46 Sep  9 10:11 1.tmp

```

### Sudo 与 Su 的区别

sudo 和 su 目标都是让用户去以其他用户的身份来执行某些操作，不同点在于：

1. sudo - u user 需要使用的是当前用户的密码
2. su - user 需要的使用的是 user 的密码

## /etc/sudoers 拆解

```bash
# 授权公式如下
授权用户/组 主机=[（切换到哪个用户/组）][是否需要输入密码验证] 命令1，命令2
# [] 中的块可以省略；多个命令之间使用, 分割；
```

### 授权用户/组

对需要授权的用户有如下两种方式：

1. 使用用户的 username 进行授权
2. 使用 User Aliases 构成用户组进行授权

```bash
# 用户名授权
username ALL=(ALL) ALL

# 构成用户组 ADMINS
User_Alias ADMINS = user1, user2, user3
# 对用户组ADMINS 授权
%ADMINS ALL=(ALL) ALL
```

### 主机

主机指的是被授权用户在哪些机器（网络位置）可以进行提权，对需要授权的主机有两种方式：

1. 直接使用 hostname, ip addresses
2. 构建主机组，进行授权

```bash
# 直接使用hostname/ip 进行授权
%ADMINS 192.168.0.1, ipa.example.com = (ALL) ALL

# 构成host 组
Host_Alias SERVERS = 192.168.0.1, 192.168.1.0/24, ipa.example.com
# 对host 组授权
%ADMINS SERVERS = (ALL) ALL
```

### 切换到哪些用户/组

该字段可以省略，**若不省略则一定要使用 () 将内容包括**，与后一字段是否需要密码验证区分。

### 是否需要输入密码

可能的值为 "NOPASSWD:"，注意后面的**冒号不能省略。**

```bash
%ADMINS SERVERS=(root:root) NOPASSWD: ALL
```

？ 有文章说 NOPASSWD: 只能影响第一个命令，后续的命令还是需要输入密码，在 rhel 7.5 中测试没有发现这种情况

### 命令

命令用逗号隔开，这些命令表示授权给当前用户以用户 A 的身份执行这些命令。

注意，一定要是用绝对路径来编写命令。（不使用绝对路径的话，用户可以在在可工作路径编写同名的脚本进行提权操作，非常危险！）

### 通配符和取消命令

example：

```bash
cdadmin ALL=(root:root) NOPASSWD: /usr/bin/*,!/usr/bin/echo
# 运行cdadmin 以root 的身份执行/usr/bin 下除了echo 的所有可执行文件
```

该处用到了通配符 \* 和取消符号!

## 配置 Sudo

sudo 的配置文件在/etc/sudoers，但是不推荐直接使用编辑器打开该文件修改，因为这样修改后系统不会进行检查是否有配置错误。

推荐使用 visudo 命令来修改该文件。

```bash
[root@SZ-licwatcher ~]# visudo
>>> /etc/sudoers: syntax error near line 101 <<<
What now?
Options are:
  (e)dit sudoers file again
  e(x)it without saving changes to sudoers file
  (Q)uit and save changes to sudoers file (DANGER!)

What now? e
```

另一种做法更为推荐，是将配置文件放到/etc/sudoers.d/ 目录下。（注意文件不能以~结尾或包含.）

1. 确认/etc/sudoers 是否有#includedir /etc/sudoers.d 这样的配置，注意# 需要保留
2. 在/etc/sudoers.d/ 目录中新建配置文件

## 其他配置

### 输入密码时反馈

在/etc/sudoers 文件中，找到 Defaults env_reset 修改为

```bash
Defaults env_reset,pwfeedback
```

### 修改 Sudo 会话时间

sudo 命令会要求输入当前用户密码，默认是 15 分钟。

如果只想第一次认证，可以修改值为 -1。
