---
title: 17.使用ocserv 在Ubuntu 20.04 上搭建SSLVPN
author: Rayd62
date: 2022-04-02 02:01:57
tags:
  - 网络技术
  - 运维
---

## Why SSLVPN & Why Ocserv
### SSLVPN
SSLVPN 是一种简单且安全的远程隧道访问技术。采用了公钥加密的方式来保障数据在传输过程中的安全性，它采用浏览器和服务器直接沟通的方式，方便了用户又通过SSL 协议来保障了数据安全。SSL 协议采用了SSL/TLS 综合加密的方法来保障数据安全。

### Ocserv
Ocserv 的主要优势有：
1. 开源，大家可以免费使用
2. 支持Cisco Anyconnect 客户端（稳定）
3. 客户端多平台支援
4. 服务器支持绝大部分Linux 和BSD
5. 支持密码认证和证书认证
6. 支持RADIUS 审计
7. 支持Virtual Hosting - 虚拟主机（Nginx - multi domain）
8. 部署简单

## 前置需求
1. Ubuntu 20.04（1G RAM）
2. 公网IP 地址
3. 域名

## 部署手册
### 1. 安装Openconnect VPN Server
使用`ssh` 登陆服务器，使用`apt`来安装`ocserv`
```bash
sudo apt update -y
sudo apt upgrade -y
sudo apt install ocserv
sudo systemctl start ocserv
```

安装完成后，使用命令检查服务状态：
```bash
sudo systemctl status ocserv
```
示例：
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203232124865.png)

## 2. 安装Let's Encrypt Client（certbot）
使用`Let's Encrypt`为VPN 服务器颁发证书。
```bash
# install Let's Encrypt Client - certbot
sudo apt install certbot

# check the version
certbot --version
# simple output: certbot 0.40.0

# open firewall, prepare for next step
ufw allow 80,443/tcp
```

## 3. 通过Let‘s Encrypt 获取可信任TLS 证书
主要有两种方式（`standalone`和`webroot`）为`ocserv`获取一个TLS 证书。
### Standalone
如果没有网站部署在你的VPN 服务器上同时你也希望VPN Server 使用443 端口来监听远程接入，那么可以使用`standalone`插件来从Let's Encrypt 获取TLS 证书。
1. 首先在DNS 服务提供商，将VPN 服务器的公网IP 绑定到准备好的域名上
2. 在VPN 服务器上使用命令获取证书

```bash
	sudo certbot certonly --standalone --preferred-challenges http --argee-tos --email youremail@email.com -d yourdomain.vpn.example.com
# certonly: 获取证书但不安装
# --standalone：使用standalone 插件来获取证书
# --preferred-challenges http： 使用http 来检验域名，会使用80 端口，注意在上一步骤中将在服务器中将防火墙放通80/tcp 端口
# --agree-tos：同意Let's Encrypt 服务条款
# --email：用来注册账户和账号恢复的邮箱
# -d：你准备好的域名
```
如果你看到如下图的信息，那么表示你已经成功获取到了TLS 证书：
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202203232147109.png)

### 使用Webroot Plugin 获取证书
推荐在VPN服务器同时部署了网站服务的时候使用这种方法，因为Webroot插件适用于几乎所有的Web服务器，我们不需要在Web服务器中安装证书。

首先，需要在网页服务器中为域名创建一个虚拟主机（virtual Host）。
#### Nginx
用熟悉的编辑器打开`/etc/nginx/conf.d/yourdomain.vpn.example.com.conf`

将下列配置粘贴到文件中，并保存：
```ngxin
server {
      listen 80;
      server_name vpn.example.com;

      root /var/www/ocserv/;

      location ~ /.well-known/acme-challenge {
         allow all;
      }
}
```

执行下列命令：
```bash
# 创建网站服务的root 目录
sudo mkdir -p /var/www/ocserv

# 将网站的root 目录权限设置给`www-data`（nginx default user）
sudo chown www-data:www-data /var/www/ocserv -R

# 重启nginx 服务
sudo systemctl reload nginx
```

然后使用`let's encrypt`后去TLS 证书：
```bash
sudo certbot certonly --webroot --agree-tos --email youremail@example.com -d yourdomain.vpn.example.com -w /var/www/ocserv
```

## 4. 编辑VPN 服务配置文件
`ocserv`配置文件：`/etc/ocserv/ocserv.conf`

### 配置认证方式
默认情况下，`ocserv`使用PAM 组件来进行认证（也就是使用Linux 用户的账号和密码来认证），建议使用独立的账号密码作为VPN 的账号：
```bash
# 在配置文件中，注释下面一行
# auth = "pam[gid-min=1000]"
# 使用一个文件来保存 VPN 用户的账号和密码
auth = "plain[passwd=/etc/ocserv/ocpasswd]"
# 完成配置文件后，我们会使用 ocpasswd 命令来创建/etc/ocserv/ocpasswd
```

### 修改VPN 服务监听端口
```bash
# 在配置文件中，注释下面两行
#tcp-port = 443
#udp-port = 443
# 指定你希望的端口用作VPN 服务
tcp-port = 4433
udp-port = 4433
```

### 修改VPN 使用的TLS 证书
```bash
# 在配置文件中，注释下面两行
# server-cert = /etc/ssl/certs/ssl-cert-snakeoil.pem
# server-key = /etc/ssl/private/ssl-cert-snakeoil.key

# 指定证书和密钥的文件，注意将vpn.example.com 替换成你的域名
server-cert = /etc/letsencrypt/live/vpn.example.com/fullchain.pem
server-key = /etc/letsencrypt/live/vpn.example.com/privkey.pem
```

### 设置最大客户端访问数量
```bash
# 最大有128 个客户端可以连接VPN，设置为0 表示无限制
max-clients = 128
```

### 设置同一用户最大接入数量
```bash
# 同一用户最大在线设备2，设置为0 表示无限制
max-same-clients = 2 
```

### 修改默认keepalive package 发送时间
```bash
# 默认值为300s，建议改小
keepalive = 30
```

### 允许MTU 探测
```bash
# 将下行false 改为true，以允许MTU 探测
try-mtu-discovery = true
```

### 设置客户端保持空闲时间
```bash
# 如果你希望客户端不自动掉线，推荐下面两行配置
idle-timeout=1200
mobile-idle-timeout=1800
```

### 设置默认域名
```bash
default-domain = yourdomain.vpn.exmaple.com
```

### 设置接入后获取的IP 网段
```bash
# 因为很多家庭网络是用的是192.168.0.0/24 或192.168.1.0/24 推荐避开这两个段
ipv4-network = 192.168.10.0
ipv4-netmask = 255.255.255.0
```

### DNS 流量通过VPN 隧道
```bash
tunnel-all-dns = true
```

### 设置VPN 接入客户端使用的DNS 服务器
```bash
# 根据实际情况选用合适的DNS 服务器，可以是内网DNS 服务器
dns = 8.8.8.8
dns = 1.1.1.1
```

### 为个别网段设置VPN 服务器为默认网关
```bash
# 连接服务器后，客户端会将10.0.0.0/8 和172.16.0.0/12 网段的网关设置为VPN 服务器
route = 10.0.0.0/8
route = 172.16.0.0/12

# 如果要用VPN 服务器作为默认网关，取消掉下行注释
# route = default
```

### 修改配置后重启ocserv 服务
```bash
sudo systemctl restart ocserv
```

## 5. 创建VPN 账户
使用 `ocpasswd` 工具创建本地VPN 账号：
```bash
# 将下列username 替换为用户账号
sudo ocpasswd -c /etc/ocserv/ocpasswd username
```


> 若需要使用 VPN 服务器作为代理服务器访问互联网，请参考以下链接。[Set Up OpenConnect VPN Server (ocserv) on Ubuntu 20.04 with Let’s Encrypt (linuxbabe.com)](https://www.linuxbabe.com/ubuntu/openconnect-vpn-server-ocserv-ubuntu-20-04-lets-encrypt)
