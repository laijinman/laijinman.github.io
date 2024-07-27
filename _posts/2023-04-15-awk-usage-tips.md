---
layout: post
title: AWK 使用技巧
date: 2023-04-15 02:55 +0000
categories:
- 系统
- Unix/Linux
tags:
- linux
- shell
- awk
---
编写服务器脚本经常使用到 AWK，收集整理 AWK 常用操作使用技巧进行分享。

## 取指定列值

取第 N 列：
```bash
cat <<EOF | awk '{print $2}'
A 12 2019
A 11 2020
C 20 2019
B 9 2021
D 3 2022
EOF
```

取首列和尾列值：
```bash
cat <<EOF | awk '{print $1, $NF}'
A 12 2019
A 11 2020
C 20 2019
B 9 2021
D 3 2022
EOF
```

## 加上行号
```bash
cat <<EOF | awk '{print NR, $0}'
A 12 2019
A 11 2020
C 20 2019
B 9 2021
D 3 2022
EOF
```

## 条件筛查

正则匹配：
```bash
cat <<EOF | awk '/2019$/{print $2}'
A 12 2019
A 11 2020
C 20 2019
B 9 2021
D 3 2022
EOF
```

指定列正则匹配：
```bash
cat <<EOF | awk '$3 ~ /19$/ {print $2}'
A 12 2019
A 11 2020
C 20 2019
B 9 2021
D 3 2022
EOF
```

值筛查：
```bash
cat <<EOF | awk '$3 == 2019 {print $2}'
A 12 2019
A 11 2020
C 20 2019
B 9 2021
D 3 2022
EOF
```

行筛查：
```bash
cat <<EOF | awk 'NR == 1 || NR == 3 {print $2}'
A 12 2019
A 11 2020
C 20 2019
B 9 2021
D 3 2022
EOF
```

## 求和/求平均值

求和：
``` bash
cat <<EOF | awk '{sum += $2} END{print sum}'
A 12 2019
A 11 2020
C 20 2019
B 9 2021
D 3 2022
EOF
```

求平均值：
``` bash
cat <<EOF | awk '{sum += $2} END{print sum/NR}'
A 12 2019
A 11 2020
C 20 2019
B 9 2021
D 3 2022
EOF
```

## 参考
<https://wangchujiang.com/linux-command/c/awk.html>

