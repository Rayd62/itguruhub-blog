---
title: 59.Bash - Read 交互
author: Rayd62
date: 2023-06-02 12:39:01
tags:
  - Linux
  - Bash
---

# Shell 交互

## Read 命令

默认接收键盘输入，回车符代表输入结束

-p: 提示消息

-t : timeout

-s: slent mode, 不回显（适用于密码输入）

-n: 输入字符个数 (超过限定字符或遇到分隔符就自动执行完而不用等待换行符)

## Example

```bash
echo -n "输入 10 个字符"
read -n10 t1
echo
echo $t1

---

kdhfbenth8
kdhfbenth8
```
