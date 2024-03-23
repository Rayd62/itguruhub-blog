---
title: 63.Ansible Facts
author: Rayd62
date: 2023-06-06 13:08:00
tags:
  - Automation
  - Ansible
---

# Ansible Facts

## Ansible Facts

用来采集被控端的信息：IP、主机名、CPU 等等

### Fact 采集失败

请确保必要的 package 在被控端已安装。

- Linux Network fact gathering - Depends on the `ip` binary, commonly included in the `iproute2` package.

### 测试

```bash
ansible 192.168.3.161 -m setup -a "filter=ansible_memtotal_mb"
# ansible host -m model -a attr
# setup 模块
# ansible_memtotal_mb 是fact 中预定义的变量，若使用filter 参数，则返回全部预定义变量。
```

## 案例 1: 通过 Facts 配置不同的 Zabbix 客户机

```yaml
- hosts: all
  tasks:
    - name: install zabbix agent
      yum:
        name: zabbix-agent
        status: present

    - name: configure Zabbix Agent
      template:
        src: ./zabbixd.conf.j2
        dst: /etc/zabbix/zabbixd.conf
        backup: yes
```

```bash
# zabbix.conf 中将Hostname 改为变量
Hostname = {{ ansible_fqdn }}
```

效果是所有根据被控端的主机名修改对应的 zabbix agent 配置文件。

## 案例 2： 根据每个主机当前 Free 的内存值的一半配置 Memcached 服务

```yaml
- hosts: all
  vars:
    package: memcached
  tasks:
    - name: install {{ package }}
      yum:
        name: '{{ package }}'
        state: present

    - name: configure {{ package }}
      template:
        src: ./configures/memcached.j2
        dest: /etc/sysconfig/memcached
        backup: yes

    - name: systemctl {{ package }} and enable it
      systemd:
        name: '{{ package }}'
        enabled: yes
        state: started

    - name: check {{ package }} status
      shell:
        cmd: systemctl status {{ package }}
      register: status

    - name: output memcached status
      debug:
        msg:
          - 'commands: systemctl status {{ package }}'
          - '{{ status.stdout_lines }}'

    - name: check memcached config file
      shell:
        cmd: cat /etc/sysconfig/memcached
      register: mem_conf

    - name: output memcached config file
      debug:
        var: mem_conf.stdout_lines
```

```bash
#./configure/memcached.conf.j2
PORT="11211"
USER="memcached"
MAXCONN="1024"
# 计算当前内存值的一半
CACHESIZE="{{ ansible_memory_mb.real.free //2 }}"
OPTIONS="-l 127.0.0.1,::1"
```

output:

```bash
[root@ray ansible]# ansible-playbook 02.memcache_installation.yml

PLAY [all] ***********************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************
ok: [192.168.3.163]
ok: [192.168.3.162]
ok: [192.168.3.161]

TASK [install memcached] *********************************************************************************************************************************************************
ok: [192.168.3.162]
ok: [192.168.3.161]
ok: [192.168.3.163]

TASK [configure memcached] *******************************************************************************************************************************************************
ok: [192.168.3.162]
ok: [192.168.3.161]
ok: [192.168.3.163]

TASK [systemctl memcached and enable it] *****************************************************************************************************************************************
ok: [192.168.3.161]
ok: [192.168.3.163]
ok: [192.168.3.162]

TASK [check memcached status] ****************************************************************************************************************************************************
changed: [192.168.3.161]
changed: [192.168.3.163]
changed: [192.168.3.162]

TASK [output memcached status] ***************************************************************************************************************************************************
ok: [192.168.3.163] => {
    "msg": [
        "commands: systemctl status memcached",
        [
            "* memcached.service - Memcached",
            "   Loaded: loaded (/usr/lib/systemd/system/memcached.service; enabled; vendor preset: disabled)",
            "   Active: active (running) since Sun 2020-10-18 01:04:29 CST; 8min ago",
            " Main PID: 4142 (memcached)",
            "    Tasks: 6",
            "   CGroup: /system.slice/memcached.service",
            "           `-4142 /usr/bin/memcached -u memcached -p 11211 -m 770 -c 1024 -l 127.0.0.1,::1",
            "",
            "Oct 18 01:04:29 localhost.localdomain systemd[1]: Started Memcached."
        ]
    ]
}
ok: [192.168.3.162] => {
    "msg": [
        "commands: systemctl status memcached",
        [
            "* memcached.service - Memcached",
            "   Loaded: loaded (/usr/lib/systemd/system/memcached.service; enabled; vendor preset: disabled)",
            "   Active: active (running) since Sun 2020-10-18 01:04:29 CST; 8min ago",
            " Main PID: 4207 (memcached)",
            "    Tasks: 6",
            "   CGroup: /system.slice/memcached.service",
            "           `-4207 /usr/bin/memcached -u memcached -p 11211 -m 768 -c 1024 -l 127.0.0.1,::1",
            "",
            "Oct 18 01:04:29 localhost.localdomain systemd[1]: Started Memcached."
        ]
    ]
}
ok: [192.168.3.161] => {
    "msg": [
        "commands: systemctl status memcached",
        [
            "* memcached.service - Memcached",
            "   Loaded: loaded (/usr/lib/systemd/system/memcached.service; enabled; vendor preset: disabled)",
            "   Active: active (running) since Sun 2020-10-18 01:04:29 CST; 8min ago",
            " Main PID: 14458 (memcached)",
            "    Tasks: 6",
            "   CGroup: /system.slice/memcached.service",
            "           `-14458 /usr/bin/memcached -u memcached -p 11211 -m 441 -c 1024 -l 127.0.0.1,::1",
            "",
            "Oct 18 01:04:29 localhost.localdomain systemd[1]: Started Memcached."
        ]
    ]
}

TASK [check memcached config file] ***********************************************************************************************************************************************
changed: [192.168.3.163]
changed: [192.168.3.161]
changed: [192.168.3.162]

TASK [output memcached config file] **********************************************************************************************************************************************
ok: [192.168.3.163] => {
    "mem_conf.stdout_lines": [
        "PORT=\\"11211\\"",
        "USER=\\"memcached\\"",
        "MAXCONN=\\"1024\\"",
        "CACHESIZE=\\"683\\"",
        "OPTIONS=\\"-l 127.0.0.1,::1\\""
    ]
}
ok: [192.168.3.162] => {
    "mem_conf.stdout_lines": [
        "PORT=\\"11211\\"",
        "USER=\\"memcached\\"",
        "MAXCONN=\\"1024\\"",
        "CACHESIZE=\\"680\\"",
        "OPTIONS=\\"-l 127.0.0.1,::1\\""
    ]
}
ok: [192.168.3.161] => {
    "mem_conf.stdout_lines": [
        "PORT=\\"11211\\"",
        "USER=\\"memcached\\"",
        "MAXCONN=\\"1024\\"",
        "CACHESIZE=\\"440\\"",
        "OPTIONS=\\"-l 127.0.0.1,::1\\""
    ]
}

PLAY RECAP ***********************************************************************************************************************************************************************
192.168.3.161              : ok=8    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.3.162              : ok=8    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.3.163              : ok=8    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Cache Plugins - 缓存插件

默认的缓存插件是将 ansible 运行时收集的 facts 数据存储在内存中，playbook 运行完立即释放。

还有缓存插件可以将 ansible 运行时收到的 facts 数据保存到文件或数据库中。

### 设置 cache_plugin

1. 环境变量的方式：

   `export ANSIBLE_CACHE_PLUGIN=jsonfile`

2. 在  `ansible.cfg`  文件中配置：

   ```bash
   [defaults]
   fact_caching=redis
   ```

## Disable Facts

在 playbook 中添加

```yaml
- hosts: all
  gather_facts: no
```

## Magic Variables

Magic 变量是预定义的变量。

Magic 变量是用来获取 ansible 的运作，包括所用 python 的版本；inventory 中的主机和组；playbook 和 roles 所在目录等等。

### 常用变量

`hostvars`, `groups`, `group_names`, `inventory_hostname`

#### 示例

如果配置数据库服务器要使用其他被控端的 fact ，或其它被控端在 inventory 中定义的变量。我们就可以在 template 中使用  `hostvars` 。

`{{ hostvars['webserver01.example.com']['ansible_facts']['distribution'] }}`

如果是针对 inventory 中的某个组，可以遍历在组中的所有主机。

```python
{% for host in group['app_servers'] %}
    # something that applies to all app servers.
{% endfor %}
```

可以将  `groups`  和  `hostvars`  组合使用来获取某个组中所有主机的 ip 地址。

```python
{% for host in groups['app_servers'] %}
    {{ hostvars[host]['ansible_facts']['eth0']['ipv4']['address'] }}
{% endfor %}
```
