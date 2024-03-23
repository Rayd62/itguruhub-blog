---
title: 58.Bash if 条件判断语句
author: Rayd62
date: 2023-06-01 14:27:36
tags:
  - Linux
  - Bash
---

# If 条件判断语句

使用  `man test`  查看条件判断文档。

## 单 if 语句

```bash
if [ condition ]
	then
		command
fi
```

example:

```bash
#!/usr/bin/bash
##############################
#Author:        Ray Ding
#Created Time:  2020-10-07_21:12:19
#Version:       1.0
#Description:
#    if current_user is not root, print error message
##############################

if [ $USER != "root" ]
    then
        echo "ERROR: need root to exec"
        exit 1
fi
```

```bash
[ray@ray ~]$ bash 06.root_con.sh
ERROR: need root to exec
```

## If-then-else 语句

两步判断，条件为真执行 A，条件为假执行 B

```bash
if [ condition ]
		then
				command A
else
				command B
fi
```

## If 嵌套

```bash
#!/usr/bin/bash
##############################
#Author:        Ray Ding
#Created Time:  2020-10-07_22:38:04
#Version:       1.0
#Description:
#    compare two integer
##############################
if [ $1 -eq $2 ]
then
    echo "$1 = $2"
else
    if [ $1 -gt $2 ]
    then
        echo "$1 > $2"
    else
        echo "$1 < $2"
    fi
fi
```

## If-elif-else

```bash
#!/usr/bin/bash
##############################
#Author:        Ray Ding
#Created Time:  2020-10-07_22:39:48
#Version:       1.0
#Description:
#    compare two integer
##############################
if [ $1 -eq $2 ]
    then
        echo "$1 = $2"
elif [ $1 -gt $2 ]
    then
        echo "$1 > $2"
else
    echo "$1 < $2"
fi
```

## (()) 和\[\[]]

在 (()) 中，可以进行数值运算

在\[\[]] 中，可以进行字符串匹配 (通配符)

```bash
#!/usr/bin/bash
##############################
#Author:        Ray Ding
#Created Time:  2020-10-07_22:48:02
#Version:       1.0
#Description:
#
##############################

if (( 100%3 + 10 > 10 ));then
    echo "yes"
else
    echo "no"
fi
```

```bash
#!/usr/bin/bash
##############################
#Author:        Ray Ding
#Created Time:  2020-10-07_22:49:17
#Version:       1.0
#Description:
#
##############################
for i in cfd ffd saq ccf wa
do
    if [[ $i == c* ]];then
        echo $i
    fi
done
```
