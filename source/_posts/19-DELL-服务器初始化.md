---
title: 19.DELL 服务器初始化
author: Rayd62
date: 2022-05-02 18:02:23
tags:
  - 运维
  - 服务器
---

# 服务器安装
1. 在服务器上，前面板配置iDRAC 地址
2. 在网页上使用该地址打开iDRAC 管理页面
        ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201411553.png)
3. 使用iDRAC 默认账号: root 默认密码: calvin 登录系统，首次登陆会提示修改iDRAC 密码
4. 首页右下角有HTML 版本的控制台，点击打开
    ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201412623.png)
5. 如果需要进行RAID 阵列卡配置，首先需要修改BIOS 启动配置，将其从UEFI 更改为BIOS
6. 保存BIOS 配置，重启系统，根据提示按ctrl + r 进入阵列卡配置界面，按需进行配置
    ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201412277.png)
    有直通需求的硬盘需要在PD Mgmt 中修改为Non-RAID
    
7. 修改完配置，点击控制台软件，点击引用键盘宏"ctrl+alt+del" 重启机器
    ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201412930.png)
8. 挂载虚拟截至（系统的ISO 文件），并在启动中将启动项修改为虚拟CD/DVD/ISO
    ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201412670.png)
9. 重启后注意查看启动是否为虚拟DVD，直通硬盘是否正常被识别。
    ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201412013.png)
10. 等待一段时间，正式进入系统安装界面，下面是RHEL 7.6 的安装步骤。
11. 选择语言
12. 配置系统时间
    ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201413363.png)
13. 设置语言英文+中文的语言支持
    ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201413461.png)
14. 配置系统盘分区
    1. 选择系统盘，选择手动分区，选择“Done"
        ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201413260.png)
    2. 删除可能存在的已有分区，按照应用组要求，进行相应的系统盘分区
        ![](https://cdn.jsdelivr.net/gh/Rayd62/note_images/202204201413540.png)
    3. 完成配置后，选择”Done“ 完成配置
15. 取消kdump
16. 开始安装，进行root 账号配置
17. 等待安装完成