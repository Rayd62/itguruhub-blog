---
title: 57.Bash 数组
author: Rayd62
date: 2023-06-01 14:27:21
tags:
  - Linux
  - Bash
---

# 数组

## 基本数组

```bash
# 定义数组
数组名称=(元素1 元素2 元素3 ...)

# 读取数组
${数组名称[索引]}

# 数组赋值（单个）
数组名称[索引]=值

# 数组赋值（多个）
数组名称=(元素1 元素2 元素3 ...)
Array1=(`cat /etc/passwd`) # 将文件中每一行作为一个元素赋值给数组
Array2=(`ls /var/ftp/shell/for`)

# 删除数组元素
unset Array_name[index]

# 删除数组
unset Array_name
```

### 查看数组

```bash
# 查看系统声明数组
declare -a
# 访问数组
${数组名[索引]}
# 访问数组的所有元素 == echo ${Array1[*]}
echo ${Array1[@]}
# 统计数组元素个数
echo ${#Array1[@]}
# 统计数组索引
echo ${!Array1[@]}
# 从索引n 开始
echo ${Array1[@]:n}
# 从索引n 开始取出m 个元素
echo ${Array1[@]:n:m}
```

### Example

```bash
  1 #!/usr/bin/bash
  2 ##############################
  3 #Author:        Ray Ding
  4 #Created Time:  2020-09-30_23:23:03
  5 #Version:       1.0
  6 #Description:
  7 # Array practise
  8 ##############################
  9 # define a new array named CHAR_ARR, and assign four value to it.
 10 CHAR_ARR=("A" "B" "C" "D")
 11 # output its all value
 12 echo ${CHAR_ARR[@]}
 13
 14
 15 # assign new value to this array
 16 echo "change index 3 value to d CHAR_ARR[3]='d'"
 17 CHAR_ARR[3]="d"
 18 echo "assign index 4 value to e"
 19 CHAR_ARR[4]="e"
 20 echo "assign index 5 value to f"
 21 CHAR_ARR[5]="f"
 22 echo "assign index 6 value to g"
 23 CHAR_ARR[6]="g"
 24
 25 echo -n "1st value: "
 26 echo ${CHAR_ARR[0]}
 27 echo -n "3rd value: "
 28 echo ${CHAR_ARR[3]}
 29
 30 echo -n "all values: "
 31 echo ${CHAR_ARR[@]}
 32
 33 echo -n "total num of value: "
 34 echo ${#CHAR_ARR[@]}
 35
 36 echo -n "array index value: "
 37 echo ${!CHAR_ARR[@]}
 38
 39 echo -n "output array value from index 3: "
 40 echo ${CHAR_ARR[@]:3}
 41
 42 echo -n "output array 2 values from index 3 : "
 43 echo ${CHAR_ARR[@]:3:2}
```

Output:

```bash
A B C D
change index 3 value to d CHAR_ARR[3]='d'
assign index 4 value to e
assign index 5 value to f
assign index 6 value to g
1st value: A
3rd value: d
all values: A B C d e f g
total num of value: 7
array index value: 0 1 2 3 4 5 6
output array value from index 3: d e f g
output array 2 values from index 3 : d e
```

### 关联数组

关联数组必须使用 declare -A 申请。

```bash
#!/usr/bin/bash
##############################
#Author:        Ray Ding
#Created Time:  2020-10-01_08:59:08
#Version:       1.0
#Description:
#    Associative arrays
#    key-value array ( I treat it as dict in python)
##############################
# declare a new array (dict)
declare -A COLOR=(["red"]="#ff0000"  ["green"]="#00ff00")

echo ${COLOR[@]}
echo ${COLOR[*]}

# add a new key-value pair/modify a key-value

COLOR["blue"]="#0000ff"

# get all values
echo "get all values:"

for value in ${COLOR[@]}
do
    echo $value
done

# get all keys
echo "get all keys:"

for key in ${!COLOR[@]}
do
    echo $key
done

# get all key - values
echo "get all key - values:"

for key in ${!COLOR[@]}
do
    echo "$key ---> ${COLOR[$key]}"
done
```
