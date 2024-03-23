---
title: 61.Ansible 基础
author: Rayd62
date: 2023-06-05 16:16:29
tags:
  - Automation
  - Ansible
---

# Ansible 基础

## Ansible 执行过程

1. 加载配置文件`/etc/ansible/ansible.cfg`
2. 加载对应的模块文件（commands）
3. 通过 ansible 将模块或命令生成对应的临时 py 文件，并将文件传输至被控端的服务器的执行用户的 `$HOME/.ansible/tmp/ansible-tmp-num/xxx.py`
4. 给执行文件 +x
5. 执行并返回结果
6. 删除临时 py 文件，sleep 0 退出

### Ansible 执行状态

1. 绿色，成功且未进行改动
2. 黄色，成功对被控端进行修改
3. 红色，未成功

## Ansible Playbook

Playbook 由 play 和 task 组成.

Play 定义了谁（要对谁/主机做操作），一个 playbook 中可以有一个或多个 play。

Task 定义了动作（怎么做/做什么操作）

Ansible Playbook 使用了 yaml 语言，yaml 语言的使用特点：

1. 除了以 ":" 冒号结尾的行，所有冒号 ":" 后都要跟一个/多个空格。注意**yaml 中一定不能出现 tab （制表符）**。
2. 所有 YAML 文件，都支持 " —-" 开始和 "..." 结束。
3. YAML 使用缩进来区分块的
4. 可以使用 "|" 或 ">" 将一行内容分散到多行
5. " #" 空格 +# 号是注释行
6. 单引号和双引号的区别是，双引号中可以使用转义符
7. ansible 使用 "{{ var }}" 作为变量，如果一个键值需要使用变量，必须用双引号标记

### Hosts and Users

每一个 play 都需要选择你的目标是哪些主机，你想使用的远端用户是哪一个。

```yaml
---
- hosts: webservers
  remote_user: root
```

remote_user 也可以在每一个 task 里面单独定义：

```yaml
---
- hosts: webservers
  remote_user: root
  tasks:
    - name: test connection
      ping:
      remote_user: xingyuan
```

如果所有的命令都想通过非 root 用户执行 (become 意思是 sudo 执行？)：

```yaml
---
- hosts: webservers
  remote_user: xingyuan
  become: yes
```

```yaml
---
- hosts: webservers
  remote_user: xingyuan
  tasks:
    - service:
        name: nginx
        state: started
      become: yes
      become_method: sudo
```

如果需要给 sudo 命令输入密码，可以使用 option -K 来执行。

### Tasks

每一个 play 都包含一个 tasks 列表。Ansible 按照 tasks 列表中的顺序依次执行。

当执行一个 playbook 时，任务失败的主机将从整个剧本的轮换中删除。

task 列表中的每一个 task 的目的就是以一些指定的命令/参数执行一个模块 (module)。变量可以在这些参数中使用。

模块有一个重要的标准就是幂等，意思是说，按顺序运行模块多次和只运行一次具有相同的效果。实现幂等的一个方法就是让模块能够检查是否已达到其所需的最终状态，如果已经达到了该状态，则不执行任何操作即可推出。如果所有的模块都是幂等的，那么使用这些模块的 playbook 也就是幂等的。

每个 task 都应该有一个 name，当 playbook 运行到这个 task 时，其值会显示到屏幕上。

#### Notify Action

notify 动作可以在 task 列表（task 块/task 组）执行完成后被执行，即使在一个 task 列表中的不同 task 语句里都包含 notify 动作，也只会在最末尾被执行一次。

notify 的目的就是仅对配置文件修改的服务进行重启，避免不必要的服务重启。

例如多个资源都指出需要重启 apache 服务，因为它们修改了 apache 的配置文件，但是 apache 只有在所有任务执行完毕并确认配置文件已修改的情况下才会重启。

```yaml
- name: template configuration file
  template:
    src: template.j2
    dest: /etc/foo.conf
  notify:
    - restart memcached
    - restart apache
```

如上面示例中的 notify 就被成为 handler。Handlers 是 task 列表（task list），它与别的任务 tasks 没有什么实质上的不同，但是它会被一个全局唯一的名称引用，并由通知者通知。

如果没有 task 引用它，那么它不会运行。但是如果有任何 task 引用它，无论有多少 task 引用它，它都会在所有 task 执行完毕后运行一次。

```yaml
handlers:
  - name: restart memcached
    service:
      name: memcached
      state: restarted
  - name: restart apache
    service:
      name: apache
      state: restarted
```

在 Ansible 2.2 后，handlers 可以通过 "listen" 关键字来产生 topic，而 task 可以使用 notify 来关联这些 topic，来使用对应的 notify 执行动作。

```yaml
handlers:
  - name: restart memcached
  - service:
      name: memcached
      state: restarted
    listen: 'restart web service'
  - name: restart apache
    service:
      name: httpd
      state: restarted
    listen: 'restart web service'
tasks:
  - name: restart web stack
    command: echo "this task will restart web services stack"
    notify: 'restart web service'
```

### Ansible-Pull

### Ansible Linting Playbook

使用 ansible-link 可以检查 yaml 文件的格式是否正确。

```bash
$ ansible-lint verify-apache.yml
[403] Package installs should not use latest
verify-apache.yml:8
Task/Handler: ensure apache is at the latest versi
```

另一种检查 yaml 语法是否正确的方式是使用 `ansible-playbook —syntax-check`  命令。

### 查看详细的执行过程

`—verbose option`  需要被添加

### 检查哪些机器会被影响

`ansible-playbook playbook.yml —list-hosts`
