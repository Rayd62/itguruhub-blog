---
title: 18.开启PVE 宿主机的nested虚拟化
author: Rayd62
date: 2022-04-02 02:09:41
tags:
  - 运维
  - Proxmox VE
  - 虚拟化
---

最近在使用Proxmox VE 平台为服务器提供虚拟化能力，顺便把EVE-NG 迁移到该平台，然后发现需要额外做一点工作来将CPU 的虚拟化功能expose 到Guest OS。以下是找到的相关资料和我在服务器上的操作记录整合。

# 检查宿主机CPU 是否支持虚拟化
检查PVE 宿主机是否开启嵌套虚拟化

```bash
root@proxmox:~# cat /sys/module/kvm_intel/parameters/nested   
N
```

显示`N`即尚未开启。进行如下操作开启嵌套虚拟化。

## 对于Intel CPU 来说（自行查看CPU 是否支持）

```bash
echo "options kvm-intel nested=Y" > /etc/modprobe.d/kvm-intel.conf
```

## 对于AMD CPU来说（自行查看CPU 是否支持）

```bash
echo "options kvm-amd nested=1" > /etc/modprobe.d/kvm-amd.conf
```

针对自己的硬件平台执行完上述操作后，重启或重载内核模块。

```bash
# 重载内核模块
modprobe -r kvm_intel
modprobe kvm_intel
```

推荐重启，省事。

这时候再次检查，应该显示`Y`代表已开启。

```bash
root@proxmox:~# cat /sys/module/kvm_intel/parameters/nested                    
Y
```

**Note: 注意什么时候用的是下划线**\*\*\*\*\*\*，什么时候用的是短横线\*\*\*\*\*\*\*\*。\*\*

# 开启EVE 的虚拟化

首先在PVE 的控制台或直接SSH 登陆主机，使用命令查看虚拟机。

```bash
# 注意110 是虚拟机主机ID，在网页中可直接看到
root@pve:~# qm showcmd 110 --pretty
/usr/bin/kvm \
  -id 110 \
  -name EVE-NG-PRO \
  -no-shutdown \
  -chardev 'socket,id=qmp,path=/var/run/qemu-server/110.qmp,server=on,wait=off' \
  -mon 'chardev=qmp,mode=control' \
  -chardev 'socket,id=qmp-event,path=/var/run/qmeventd.sock,reconnect=5' \
  -mon 'chardev=qmp-event,mode=control' \
  -pidfile /var/run/qemu-server/110.pid \
  -daemonize \
  -smbios 'type=1,uuid=a72029d4-c893-49c1-a9f3-cf50378eda30' \
  -smp '14,sockets=1,cores=14,maxcpus=14' \
  -nodefaults \
  -boot 'menu=on,strict=on,reboot-timeout=1000,splash=/usr/share/qemu-server/bootsplash.jpg' \
  -vnc 'unix:/var/run/qemu-server/110.vnc,password=on' \
  -cpu kvm64,enforce,+kvm_pv_eoi,+kvm_pv_unhalt,+lahf_lm,+sep \
  -m 61440 \
  -device 'pci-bridge,id=pci.1,chassis_nr=1,bus=pci.0,addr=0x1e' \
  -device 'pci-bridge,id=pci.2,chassis_nr=2,bus=pci.0,addr=0x1f' \
  -device 'vmgenid,guid=8a062fe1-5c54-4382-8ef8-e0dc13e274f8' \
  -device 'piix3-usb-uhci,id=uhci,bus=pci.0,addr=0x1.0x2' \
  -device 'usb-tablet,id=tablet,bus=uhci.0,port=1' \
  -chardev 'socket,id=serial0,path=/var/run/qemu-server/110.serial0,server=on,wait=off' \
  -device 'isa-serial,chardev=serial0' \
  -device 'VGA,id=vga,bus=pci.0,addr=0x2' \
  -device 'virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x3' \
  -iscsi 'initiator-name=iqn.1993-08.org.debian:01:dcfb1238fcc' \
  -device 'lsi,id=scsihw0,bus=pci.0,addr=0x5' \
  -drive 'file=/dev/pve/vm-110-disk-0,if=none,id=drive-scsi0,format=raw,cache=none,aio=io_uring,detect-zeroes=on' \
  -device 'scsi-hd,bus=scsihw0.0,scsi-id=0,drive=drive-scsi0,id=scsi0,bootindex=100' \
  -netdev 'type=tap,id=net0,ifname=tap110i0,script=/var/lib/qemu-server/pve-bridge,downscript=/var/lib/qemu-server/pve-bridgedown' \
  -device 'e1000,mac=A6:A9:53:54:4F:F8,netdev=net0,bus=pci.0,addr=0x12,id=net0' \
  -machine 'type=pc+pve0'
```

拷贝得到的输出到编辑器中，找到其中`-cpu kvm64,enforce,+kvm_pv_eoi,+kvm_pv_unhalt,+lahf_lm,+sep`

在其后添加`+vmx`，即：

```bash
/usr/bin/kvm \
  -id 110 \
  -name EVE-NG-PRO \
  -no-shutdown \
  -chardev 'socket,id=qmp,path=/var/run/qemu-server/110.qmp,server=on,wait=off' \
  -mon 'chardev=qmp,mode=control' \
  -chardev 'socket,id=qmp-event,path=/var/run/qmeventd.sock,reconnect=5' \
  -mon 'chardev=qmp-event,mode=control' \
  -pidfile /var/run/qemu-server/110.pid \
  -daemonize \
  -smbios 'type=1,uuid=a72029d4-c893-49c1-a9f3-cf50378eda30' \
  -smp '14,sockets=1,cores=14,maxcpus=14' \
  -nodefaults \
  -boot 'menu=on,strict=on,reboot-timeout=1000,splash=/usr/share/qemu-server/bootsplash.jpg' \
  -vnc 'unix:/var/run/qemu-server/110.vnc,password=on' \
  -cpu kvm64,enforce,+kvm_pv_eoi,+kvm_pv_unhalt,+lahf_lm,+sep,+vmx \
  -m 61440 \
  -device 'pci-bridge,id=pci.1,chassis_nr=1,bus=pci.0,addr=0x1e' \
  -device 'pci-bridge,id=pci.2,chassis_nr=2,bus=pci.0,addr=0x1f' \
  -device 'vmgenid,guid=8a062fe1-5c54-4382-8ef8-e0dc13e274f8' \
  -device 'piix3-usb-uhci,id=uhci,bus=pci.0,addr=0x1.0x2' \
  -device 'usb-tablet,id=tablet,bus=uhci.0,port=1' \
  -chardev 'socket,id=serial0,path=/var/run/qemu-server/110.serial0,server=on,wait=off' \
  -device 'isa-serial,chardev=serial0' \
  -device 'VGA,id=vga,bus=pci.0,addr=0x2' \
  -device 'virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x3' \
  -iscsi 'initiator-name=iqn.1993-08.org.debian:01:dcfb1238fcc' \
  -device 'lsi,id=scsihw0,bus=pci.0,addr=0x5' \
  -drive 'file=/dev/pve/vm-110-disk-0,if=none,id=drive-scsi0,format=raw,cache=none,aio=io_uring,detect-zeroes=on' \
  -device 'scsi-hd,bus=scsihw0.0,scsi-id=0,drive=drive-scsi0,id=scsi0,bootindex=100' \
  -netdev 'type=tap,id=net0,ifname=tap110i0,script=/var/lib/qemu-server/pve-bridge,downscript=/var/lib/qemu-server/pve-bridgedown' \
  -device 'e1000,mac=A6:A9:53:54:4F:F8,netdev=net0,bus=pci.0,addr=0x12,id=net0' \
  -machine 'type=pc+pve0' \
  -cpu kvm64,enforce,+kvm_pv_eoi,+kvm_pv_unhalt,+lahf_lm,+sep,+vmx
```

复制所得，然后在PVE 的CLI 界面使用该命令，启动VM 110 （EVE-NG）。

使用`qm list` 查看虚拟机状态，

```bash
oot@pve:~# qm list
      VMID NAME                 STATUS     MEM(MB)    BOOTDISK(GB) PID       
       110 EVE-NG-PRO           running    61440              0.00 1434 
```

显示虚拟机已经启动。

这是登陆到虚拟机的CLI 界面，使用`egrep "vmx|svm" /proc/cpuinfo flags`，查看是否成功在虚机中启动虚拟化。

```bash
root§eve-ng:ß# egrep "vmx|svm" --color=always /proc/cpuinfo      
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx lm constant_tsc nopl xtopology cpuid tsc_known_freq pni vmx cx16 x2apic hypervisor lahf_lm cpuid_fault tpr_shadow
```

# 将VMX参数添加为默认启动项

在PVE 的host 机中，编辑需要启动`vmx`的主机的配置文件。文件存放于`/etc/pve/qemu-server`下。

修改对应vm-id 的配置文件，在这里需要修改的文件是`110.conf`

在第一行添加`args: -cpu 'kvm64,enforce,+kvm_pv_eoi,+kvm_pv_unhalt,+lahf_lm,+sep,+vmx'`（注意该命令请通过`qm showcmd 110 —pretty`命令，找到CPU 选项的一行，手动添加`+vmx`），保存。通过网页启动110 虚机时无需再通过命令行手动添加`vmx`启动选项启动guest virtualization nest。

