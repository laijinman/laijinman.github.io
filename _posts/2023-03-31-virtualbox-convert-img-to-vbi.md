---
layout: post
title: VirtualBox 转换 IMG 到 VBI
date: 2023-03-31 06:39 +0000
categories: [Server, VirtualBox]
tags: [virtualbox]
---

网上下载的虚拟机镜像文件很多是`.img`格式，VirtualBox 只支持使用`.vbi`格式，通过 VirtualBox 自带的工具 VBoxManage 可以进行格式转换。


```bash
VBoxManage convertdd source.img target.vdi
```
