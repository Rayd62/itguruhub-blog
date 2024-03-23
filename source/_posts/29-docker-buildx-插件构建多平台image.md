---
title: 29.docker buildx 插件构建多平台image
author: Rayd62
date: 2022-08-27 12:52:38
tags:
  - container
  - 运维
---

# 前言
最近写了一些小工具来提高办公效率，在分享给同事的时考虑简单部署的问题（太懒了不想去调他们的环境），就直接build 成一个容器image 上传到registry 让他们自己拉。

没想到我通过M1 Mac build 出来的image 只能在arm64 v8 的平台使用，搜索了一下，通过buildx 插件可以可以在build 时选择多平台来制作镜像。
<!-- more -->

# Buildx 简介
`Docker Buildx`是一个CLI插件，它扩展了docker命令，来完全支持Moby BuildKit构建工具包提供的功能。

只需要使用命令`docker buildx build .`就可以开始使用工具包来制作镜像。

`Docker Buildx Build`命令支持可用于docker构建功能，包括输出配置、内联构建缓存和指定目标平台等功能。此外，Buildx还支持常规`docker build`命令尚未提供的新功能，如构建清单列表、分布式缓存以及将构建结果导出到OCI镜像tarball。

[Docker Officer - Working with Buildx](https://docs.docker.com/build/buildx/)

# 安装Buildx

在Windows、MacOS 和Linux 上，安装docker desktop 会包含docker buildx。

如果需要手动安装，可以查看官方手册[手动安装](https://docs.docker.com/build/buildx/install/#manual-download)

## 设置Buildx 为默认builder
运行命令`docker buildx install` 将`docker build`命令设置成`docker buildx build` 的别名，即通过使用命令`docker build` 来使用当前的buildx builder`。

如果要移除别名设置，运行命令`docker buildx uninstall`。

# Buildx Driver
Buildx客户端会连接到BuildKit后端以执行构建。Buildx 驱动程序则允许对后端的管理进行细粒度控制，例如BuildKit运行的位置、运行方式等多种选项。
目前支持的有四种：
+ docker: 直接调用docker 进程中buildkit 库
+ docker-container: 使用Docker启动了一个专用的BuildKit容器，用于访问高级功能。
+ kubernetes: 在远程Kubernetes集群中启动专用BuildKit Pod，用于可扩展构建。
+ remote: 允许直接连接到手动管理的BuildKit守护程序，以便进行更多自定义设置。

| Feature | docker | docker-container | kubernetes | remote |
| --- |---| --- | --- | --- |
| automatic `--load` | Y | N | N |N |
| cache export | (inline only) | Y | Y | Y |
| Docker/OCI tarball output | N | Y | Y | Y |
| Mutil-arch images | N | Y | Y | Y |
| BuildKit configuration | N | Y | Y | (managed externally) |

# 构建多平台镜像
使用`--platform` 参数就可以来构建多个平台的镜像了。
```bash
docker buildx build -t imageName:version --platform=linux/arm64,linux/amd64 .
```
等待镜像构建完成，使用命令`docker buildx imagetools inspect imageName:version` 就可以查看镜像支持的平台。
![](https://cdn.jsdelivr.net/gh/Rayd62/note_images02/202208271335510.png)

## ARM 系统架构标识
```bash
arm64, armv8l, arm64v8, aarch64
arm, arm32, arm32v7, armv7, armv7l, armhf
arm32v6, armv6, armv6l, arm32v5, armv5, armv5l, armel, aarch32
```
## Intel&AMD 系统架构标识
```bash
x86, 386, i386, i686
x86_64, x64, amd64
```