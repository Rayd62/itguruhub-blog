---
title: 30.Debian 解决中文乱码
author: Rayd62
date: 2022-10-09 16:53:27
tags:
  - 运维
  - Linux
---

默认只安装英文系统的主机上，中文字符会显示为乱码，这是由于内核中没有中文字符库造成的。想解决这个问题需要手动安装字符库。[详见](https://www.debian.org/doc/manuals/debian-reference/ch08.zh-cn.html)

# 安装 locales

一般该软件是在系统装机时默认安装的，否则需要手动安装 `locale`, `locales`/`locales-all` 软件包。

# 语言环境重新配置

使用命令 `dpkg-reconfigure locales` 重新配置 `locale` 语言环境。使用命令后，进入下面界面：  
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images02/202210090900285.png)  
使用 `空格` 选中 `en_US.UTF-8 UTF-8` 和 `zh_CN.UTF-8 UTF-8`（一般来说，保证有一个 UTF-8 字符库被选中即可，此处选择两个是为了保证以后其它场景后续无需再配置）

使用 `tab` 键跳转到 `ok` 进入下一步。  
提示需要为系统选择一个默认语言包，这里推荐选择 `en_US.UTF-8`。  
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images02/202210090904618.png)

等待语言包安装配置完成。

```bash
root@Nginx:~/homepage# dpkg-reconfigure locales 
Generating locales (this might take a while)...
  en_US.UTF-8... done
  zh_CN.UTF-8... done
  zh_TW.UTF-8... done
Generation complete.
```

使用命令 `locale` 查询配置结果：

```bash
root@Nginx:~/homepage# locale
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

如果显示依然不是 `UTF-8` 的语言，重启系统再查看。

