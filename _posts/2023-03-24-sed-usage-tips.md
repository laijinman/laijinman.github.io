---
layout: post
title: SED 使用技巧
date: 2023-03-24 12:37 +0000
categories: [Server, Shell]
tags: [linux, shell, sed]
---

编写服务器脚本经常需要用 SED，整理常用 SED 操作技巧收藏一下，以备不时之需。

## 简单的匹配删除

删除匹配的行（`/d`）：
```bash
cat <<EOF | sed '/^\s*key1:/d'
section1:
  key1: value1
  key2: value2
section2:
  key1: value1
  key2: value2
EOF
```

删除不匹配的行（`/d`换成`/!d`)：
```bash
cat <<EOF | sed '/^\s*key1:/!d'
section1:
  key1: value1
  key2: value2
section2:
  key1: value1
  key2: value2
EOF
```
## 简单的匹配替换

直接替换匹配的行：
```bash
cat <<EOF | sed '/#key3:/c\  key3: value3'
section1:
  key1: value1
  key2: value2
  #key3: value3
section2:
  key4=value4
EOF
```

替换匹配的行时引用匹配值：
```bash
cat <<EOF | sed 's/^\(\s*\)#\(.*\)$/\1\2/g'
section1:
  key1: value1
  key2: value2
  #key3: value3
section2:
  key4=value4
EOF
```

## 简单的增加内容

插入内容到匹配的行之前：
```bash
cat <<EOF | sed '/section1:/i\common:'
section1:
  key1: value1
  key2: value2
  #key3: value3
section2:
  key4=value4
EOF
```

添加内容到匹配的行之后：
```bash
cat <<EOF | sed '/common:/a\  key: value'
common:
section1:
  key1: value1
  key2: value2
  #key3: value3
section2:
  key4=value4
EOF
```

## 复杂的模式匹配操作

先看看通用的例子：
```bash
sed '/pattern1/,/pattern2/ {
  ...
  /pattern3/,/pattern4/ {
    ...
  }
} file
```
通过一些简单操作的无限组合，可完成非常复杂的操作。一些特殊的匹配规则如下：
- `/pattern/`：模式匹配`pattern`内容，其中`1`和`$`表示匹配首行和尾行；
- `/pattern/!`：模式不匹配`pattern`内容；
- `/pattern1/,/pattern2/{...}`：模式匹配`pattern1`和`pattern2`之间内容，`{...}`操作只在匹配内容之间有效，可以无限套娃；

删除首行到第 N 行内容：
```bash
cat <<EOF | sed '1,+2d'
section1:
  key1: value1
  key2: value2
section2:
  key1: value1
  key2: value2
EOF
```

删除匹配行到尾行内容：
```bash
cat <<EOF | sed '/section2:/,$d'
section1:
  key1: value1
  key2: value2
  #key3: value3
section2:
  key4=value4
EOF
```
也可以这样：
```bash
cat <<EOF | sed '/section2:/,$ {
  /.*/d
}'
section1:
  key1: value1
  key2: value2
  #key3: value3
section2:
  key4=value4
EOF
```
`/.*/d`可以简写为`d`。


删除两个模式匹配行之间的内容：
```bash
cat <<EOF | sed '/section1:/,/section2:/{
  /section1:/!{
    /section2:/!d
  }
}'
section1:
  key1: value1
  key2: value2
  #key3: value3
section2:
  key4=value4
EOF
```

替换两个模式匹配行之间的内容：
```bash
cat <<EOF | sed '/section1:/,/section2:/ {
  s#key1#key3#g
}'
section1:
  key1: value1
  key2: value2
section2:
  key1: value1
  key2: value2
EOF
```
也可以这样：
```bash
cat <<EOF | sed '/section1:/ {
  :a
  n
  s#key1#key3#g
  /section2:/!ba
}'
section1:
  key1: value1
  key2: value2
section2:
  key1: value1
  key2: value2
EOF
```
- `:a`表示定义一个名为`a`的跳转标签，`ba`则为跳转到`a`标签；
- `n`表示读取下一行。

删除匹配的行下一行到尾行内容：
```bash
cat <<EOF | sed '/section2:/ {
  p
  :a
  N
  $!ba
  d
}'
section1:
  key1: value1
  key2: value2
section2:
  key1: value1
  key2: value2
EOF
```
- `p`表示打印匹配行，也就是`section2:`；
- `N`表示追加下一行到模式匹配。

## 参考
<https://wangchujiang.com/linux-command/c/sed.html>
