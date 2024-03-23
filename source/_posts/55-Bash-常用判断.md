---
title: 55.Bash 常用判断
author: Rayd62
date: 2023-05-31 16:35:46
tags:
  - Linux
  - Bash
---

# 常用判断

## 参数：

```bash
$# # 多少个参数
$* # 所有参数的值
$? # 上一条语句执行的返回值（0为成功，非0 为失败）

$0 # 脚本名称
$1 # 第一个参数
```

| 符号       | 描述                                                  |
| ---------- | ----------------------------------------------------- |
| ~          | 当前用户的 Home 目录                                  |
| $          | 变量中取内容符                                        |
| + - \* / % | 加 减 乘 除 取余                                      |
| &          | 后台执行                                              |
| \*         | 通配符，匹配所有                                      |
| ?          | 通配符，匹配除回车外的一个字符                        |
| ;          | 分号在 shell 中可一行自信多条命令，命令之间用分号分隔 |
| \\         | 转义符                                                |
| \|         | 管道符，将上一条命令的执行结果作为下一条命令的输入    |
| \`\`       | 命令中执行命令 echo "Date is: \`date\`"               |
| ''         | 单引号，字符串，不能解释变量                          |
| ""         | 双引号，字符串，可以解释变量                          |

## 文件测试

```bash
[ -f /etc/fstab ] # 判断fstab 是否为文件, 注意语句与'[]' 之间必须有空格
-d # 测试文件是否为目录类型
-e # 测试文件/目录是否存在
-r # 测试当前用户是否有读取权限，root 对000 的文件也可读
-w # 测试当前用户是否有写入权限
-x # 测试当前用户是否有执行权限
-s # 测试文件是否存在且不为空
-O # 测试文件是否存在且被当前用户拥有，大写O
-G # 测试文件是否存在且被为当前用户组所有，大写G
file1 -nt file2 # 检查file1 是否比file2 新
file1 -ot file2 # 检查file1 是否比file2 旧
file1 -ef file2 # 检查file1 和file2 是否是同一个文件，相同inode 的文件为0，其他为1
```

### 逻辑符

```bash
A && B # 若命令A 执行成功，则执行命令B
A || B # 若命令A 执行失败，则执行命令B
! A    # 若命令A 成功，则返回失败（取反）

[ ! $USER = root ] && echo "Normal User" || echo "Root"
# 判断用户是否不是root ，若不是则输出Normal User，若是则输出Root
```

### 数值判断

```bash
# man test 查看文档
A -eq B # 判断A 是否等于B
A -gt B # 判断A 是否大于B
A -lt B # 判断A 是否小于B
A -ge B # 判断A 大于或等于B
A -le B # 判断A 小于或等于B
A -ne B # 判断A 不等于B
```

### 浮点数判断

浮点数先转换成整数再判断

```bash
test `echo "1.5*10"|bc|cut -d '.' -f1` -eq $((2*10));echo $?
1
```

Example:

```bash
#!/usr/bin/bash
##############################
#Author:        Ray Ding
#Created Time:  2020-10-07_20:31:15
#Version:       1.0
#Description:
#    compare floating value
##############################

NUM1=`echo "1.5*10"|bc|cut -d "." -f1`
NUM2=$((2*10))

test $NUM1 -eq $NUM2 && echo "NUM1 = NUM2" || echo "NUM1 != NUM2"
```

```bash
#output
# bash -x debug mode
[root@ray shell]# bash -x 05.float_comparison.sh
++ echo '1.5*10'
++ cut -d . -f1
++ bc
+ NUM1=15
+ NUM2=20
+ test 15 -eq 20
+ echo 'NUM1 != NUM2'
NUM1 != NUM2
```

### 字符串比较

```bash
A == B # 判断A 与B 是否一致
A != B # 判断A 与B 是否不一致
-z ""  # 判断字符串是否为空
-n ""  # 判断字符串是否不为空
```

## 重定向描述符

语法：

- ">": 文件描述符 > 文件名
- ">&": 文件描述符 >& 文件描述符
- "&>": &> 文件名

文件描述符有：

- 0: stdin
- 1: stdout
- 2: stderr

```bash
# 用法
>file_name # == 1>file_name，标准输出重定向到文件
&>file_name # == "1>file_name 2>&1", 标准输出和标准错输出重定向到文件

```

```bash
paste <(seq 1 5) <(seq 129 133) | while read host ip; do echo "vm$host: 172.17.5.$ip"; done
```
