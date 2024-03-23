---
title: 62.Ansible Variable
author: Rayd62
date: 2023-06-06 12:59:03
tags:
  - Automation
  - Ansible
---

# Ansible Variable

## 变量简介

1. 变量不能是**python keywords 或 playbook keywords**。
2. 变量由字母，数字和下划线组成，数字不能开头
3. 以下划线开头的变量并不是私有变量（ansible 中没有区分私有公有变量）

### 简单变量

即变量名 + 值

#### 定义变量

使用标准 YAML 语法来定义：

```yaml
- hosts: app_servers
  vars:
    remote_install_path: /opt/my_app_config
```

#### 使用变量

在 playbook 中，使用 jinja2 语法来引用变量。

```yaml
template: src=foo.cfg.j2 dest={{ remote_install_path }}/foo.cfg
```

### 列表变量

即变量名 + 多个值，值是有顺序（序列化的）存储在计算机中。

#### 定义变量

```yaml
region:
  - northeast
  - southeast
  - midwest
```

#### 使用变量

```yaml
region: '{{ region[0] }}'
## output: northeast
```

### 字典变量

字典变量保存 key-value 对。

#### 定义变量

```yaml
foo:
  field1: one
  field2: two
```

#### 使用变量

```yaml
foo['filed1']
foo.field2
```

尽量使用括号形式来引用，因为 dot 形式有可能会引起关键字冲突。

### 注册变量

使用 register 关键字创建变量接收 ansible task 的返回值。

注册变量可以是简单变量，列表变量或字典变量。

每一个模块的文档中包含的 RETURN 块，描述了该模块的返回值信息。

在运行 playbook 的使用可以通过添加 -v option 来检查这类变量的值。

```yaml
- hosts: web_servers
  tasks:
    - shell: /usr/bin/foo
      register: foo_result
      ignore_errors: True

    - shell: /usr/bin/bar
      when: foo_result.rc == 5
```

```yaml
# 输出shell 模块命令执行结果
- hosts: webserver
  tasks:
    - name: install httpd
      yum: name=httpd state=present

    - name: httpd server start
      service: name=httpd state=started

    - name: check httpd status
      shell: ps aux | grep httpd
      register: httpd_status

    - name: Output httpd status
      debug:
        msg: Commands: "{{ httpd_status.cmd }}"
        msg: "{{ httpd_status.stdout_lines }}"
```

### 引用嵌套变量

许多注册变量是嵌套的 YAML 或 JSON 数据。对于这些数据不能简单的通过{{ foo }} 来引用。必须使用中括号或 dot 符的形式来引用。

例如，引用 ansible_facts 这个变量中的 IP 地址：

```yaml
{{ ansible_facts["eth0"]["ipv4"]["address"] }}
```

## 通过 Jinja2 Filter 改变变量

Jinja2 filter 让人在模版表达式 (template expression) 中改变变量的值。

例如， capitalize filter 可以将所有传给它的值大写输出。

[Template Designer Documentation - Jinja Documentation (2.11.x)](http://jinja.pocoo.org/docs/templates/#builtin-filters)

## 在哪里定义变量

- inventory
- playbook
- reusable files
- roles
- command line

### 在 Inventory 中定义变量

可以对不同的 host 定义不同的变量，或设置共享变量给一组或所有 host。

#### Variable for Individual Host

```yaml
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

#### Variable for a Group of Host

```yaml
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

**注意在 inventory 中定义的使用的是“=”**

#### 继承变量：组组的组变量 (group Variables for Groups of groups)

可以在 INI 中使用 :children 后缀或在 YMAL 中使用 children: 的前缀来创建基于组的组。

```yaml
[atlanta]
host1
host2

[raleign]
host2
host3

[southeast:children]
atlanta
raleign

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

### 在 Playbook 中定义变量

直接在 playbook 中定义变量，在 hosts 下方定义该 hosts 使用的变量。

```yaml
- hosts: webservers
  vars:
    http_port: 80
```

### 在独立文件或 Roles 中定义变量

```yaml
---
- hosts: all
  remote_user: root
  vars:
    favcolor: blue
  vars_files:
    - /vars/external_vars.yml

  tasks:
    - name: this is just a placeholder
      command: /bin/echo foo
```

content of external_vars.yml

```yaml
---
# in the above example, this would be vars/external_vars.yml

somevar: somevalue
password: magic
```

### 在运行时定义变量

在执行 playbook 时，使用—extra-vars 或 -e 选项来定义变量。也可以使用 vars_prompt 来交互式的输入变量值。

#### key=value 格式

使用 key=value 的格式，并使用“”将内容包括。

使用这种格式，解**释器会将内容当作字符串类型**；如果需要输入的变量值是布尔型，整数，浮点型，列表等等，应该使用 JSON 格式来输入变量。

```bash
ansible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"
```

#### JSON 格式

```bash
ansible-playbook release.yml --extra-vars '{"version":"1.23.45","other_variable":"foo"}'
ansible-playbook arcade.yml --extra-vars '{"pacman":"mrs","ghosts":["inky", "pinky","clyde","sue"]}'
```

#### JSON or YAML File

如果希望将一个 JSON 或 YAML 文件作为值传入：

```bash
ansible-playbook release.yml --extra-vars "@some_file.json"
```

## Variables 优先级

[Using Variables - Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#tips-on-where-to-set-variables)

### 实例

```yaml
# 安装httpd 并启动
- hosts: webserver
  vars:
    package_1: httpd
  tasks:
    - name: install {{ package_1 }}
      yum:
        name: '{{ package_1 }}'
        state: present

    - name: start {{ package_1 }}
      service:
        name: '{{ package_1 }}'
        state: started

    - name: add firewall rule
      firewalld:
        service: http
        permanent: yes
        state: enabled
        zone: public
        immediate: yes

    - name: check {{ package_1 }} status
      shell: ps ax | grep {{ package_1 }} | grep -v grep
      register: package_1_status

    - name: output httpd status
      debug:
        msg:
          - "Command: '{{ package_1_status.cmd }}'"
          - '{{ package_1_status.stdout_lines }}'
```

**注意该实例中有的变量使用了双引号有的没有，一定要切记！**
