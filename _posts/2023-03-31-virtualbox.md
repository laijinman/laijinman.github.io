---
layout: post
title: VirtualBox 虚拟化工具使用
date: 2023-03-31 06:39 +0000
categories:
  - 技术
tags:
  - virtualbox
---
 
不是 VirtualBox 的使用和入门教程，主要是收集 VirtualBox 及附带的相关工具在日常学习和工作时，解决实际遇到的问题的操作方法。

## 镜像格式转换

VirtualBox 自带的 VBoxManage 工具可以对镜像进行格式转换，操作如下：

```bash
VBoxManage convertdd from.img to.vdi
```
