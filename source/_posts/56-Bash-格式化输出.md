---
title: 56.Bash 格式化输出
author: Rayd62
date: 2023-05-31 16:35:55
tags:
  - Linux
  - Bash
---

# 格式化输出

## Echo 命令

功能：将内容输出到默认显示设备

```bash
echo [options] string
```

echo 会将输入的字符串送往标准输出。输出字符串间以空白符隔开。并在最后加上换行符。

### 选项

-n: 末尾不加换行符。

-e: 转义字符

```
   \\\\      反斜杠

   \\\\a     alert (BEL)，提示音

   \\\\b     backspace

   \\\\c     produce no further output

   \\\\e     escape

   \\\\f     form feed

   \\\\n     new line

   \\\\r     carriage return

   \\\\t     horizontal tab

   \\\\v     vertical tab

   \\\\0NNN  byte with octal value NNN (1 to 3 digits)

   \\\\xHH   byte with hexadecimal value HH (1 to 2 digits)
```

### 字体颜色、背景色

```bash
echo -e "\\033[背景色;字体颜色m 字符串 \\[属性效果m"
```

| 颜色   | 字体 | 背景 |
| ------ | ---- | ---- |
| 黑     | 30   | 40   |
| 红     | 31   | 41   |
| 绿     | 32   | 42   |
| 黄     | 33   | 43   |
| 蓝     | 34   | 44   |
| 品红   | 35   | 45   |
| 蓝绿   | 36   | 46   |
| 白     | 37   | 47   |
| 灰色   | 90   | 100  |
| 亮红   | 91   | 101  |
| 亮绿   | 92   | 102  |
| 亮黄   | 93   | 103  |
| 亮蓝   | 94   | 104  |
| 亮品红 | 95   | 105  |
| 亮蓝绿 | 96   | 106  |
| 亮白   | 97   | 107  |
