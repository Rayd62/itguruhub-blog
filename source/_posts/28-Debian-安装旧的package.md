---
title: 28.Debian 安装旧的package
author: Rayd62
date: 2022-08-10 16:55:19
tags:
  - 运维
  - Linux
---
如果需求旧的依赖包，但是在最新的apt source 中已经无法找到。那么可以去[snapshot.debian.org](https://snapshot.debian.org/) 寻找对应的archive 来安装

## 示例
需要在Debian 9 中安装`linux-headers-4.9.0-14-amd64`。但是最新的source-list 只有`linux-headers-amd64/oldoldstable 4.9+80+deb9u17 amd64`，对特殊软件来说高版本不一定能够兼容。

那么前往[snapshot.debian.org](https://snapshot.debian.org/) 搜索`linux-headers-4.9.0-14-amd64`
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images02/202208061645358.png)
查找到`linux-headers-4.9.0-14-amd64`最新的子版本：
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images02/202208061645834.png)
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images02/202208061646492.png)

可以看到在2022-12-17 23:01:51 的归档中有此版本的安装包。

这里是`debian-security` 所以前往[debian-security - snapshot.debian.org](https://snapshot.debian.org/archive/debian-security/) 中去查找这个归档。

> 若是`debian` 的分支，则前往[snapshot.debian.org](https://snapshot.debian.org/archive/debian/) 中去查找归档。

定位该归档：
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images02/202208061648234.png)
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images02/202208061649532.png)
点击这个归档，然后将url 复制。

在`Debian 9` 中，前往`/etc/apt/source.list` 目录，创建`snapshot.list` 文件，在其中添加：
```bash
deb http://snapshot.debian.org/archive/debian-security/20201217T230151Z stretch/updates main
```
注意，此处https 改为http，另外末尾的`/` 删除。

保存，在root 权限下执行`apt -o Acquire::Check-Valid-Until=false update`，这条命令能用Valid-Until来访问超过12天的套件快照，有必要忽略Release文件中的Valid-Until标头，以防止APT忽略快照条目(“Release Files Expired”)。

然后使用命令`apt-get install linux-headers-4.9.0-14-amd64`，执行成功。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images02/202208061654873.png)


```bash
deb http://snapshot.debian.org/archive/debian/20201220T203231Z stretch main
deb http://snapshot.debian.org/archive/debian/20201220T203231Z stretch-updates main
deb http://snapshot.debian.org/archive/debian-security/20201217T230151Z stretch/updates main
```