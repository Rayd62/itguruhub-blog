---
title: 32.Nginx - 反代、双向认证
author: Rayd62
date: 2022-11-06 10:36:07
tags:
  - Nginx
  - 运维
---

# 前言

想在公网访问家里部署的几个服务，VPN 连接家庭网络的方案有时会影响到公司内网的访问，主要是 DNS 的一些问题。最后想想还是将服务发布到公网，因为只有自己一个人使用，所以弄了一台 Nginx 服务器作为服务的前置代理，并在上面配置了认证客户端证书相关配置，满足了公网访问的安全性需求。

<!-- more -->

# Nginx 反向代理

实现效果：访问 `www.hf2.life:1329`，跳转到后台的 `10.1.1.60:1329`

## 配置代码

```nginx
server {
  listen 1329;
  server_name www.hf2.life;

  # 其他配置
  # ...
  # ...

  # proxy 配置
  location / {
    root html;
    index index.html index.htm;
    proxy_pass http://10.1.1.60:1329;
  }
}
```

# Nginx 配置客户端证书认证

## 使用 openssl 生成证书

```bash
// 生成key
openssl genrsa -des3 -out Client.key 4096

// 根据key 生成 pem 文件
openssl req -x509 -new -nodes -key Client.key -sha256 -days 730 -out Client.pem
```

进入 `openssl`，将 `pem` 文件转换为 `p12` 文件方便客户端导入

```bash
> openssl
OpenSSL> pkcs12 -export -in Client.pem -inkey Client.key -out Client.p12
Enter Export Password:
Verifying - Enter Export Password:

// 建议配置密码
```

## 服务配置

将上面过程生成的 `pem` 文件上传到 `nginx` 服务器，修改 `nginx` 配置。

```nginx
server {
# Client SSL Setting
  ssl_client_certificate /usr/share/CA/Client.pem;
  ssl_verify_client on;
  }
```

# 客户端浏览器配置

## Firefox 导入证书

打开 `选项` - `设置` - `隐私与安全` - `证书` - `查看证书`。  
选择`您的证书`，点击`导入`，选中上面步骤生成的`p12` 文件，输入密码，导入成功。

## IOS 导入证书

在电脑上开启一个 http 服务器，用 iphone safari 浏览器打开页面，下载上面步骤生成的 `p12` 文件。

按提示，打开 `设置` - `已下载描述文件` - ` 安装`，按提示步骤操作即可。
