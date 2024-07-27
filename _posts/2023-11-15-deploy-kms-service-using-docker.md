---
layout: post
title: Docker 方式部署 KMS 服务
date: 2023-11-15 09:33 +0000
categories: 
- 系统
- Windows
tags: 
- docker
- vlmcsd
- kms
---
通过 Docker 方式部署 KMS 服务和简单使用介绍。

## 部署 KMS 服务
```yml
version: "3"

networks:
  home:
    name: home

services:
  vlmcsd:
    image: vlmcsd/vlmcsd
    read_only: true
    container_name: vlmcsd
    hostname: vlmcsd.home
    networks:
    - home
    restart: unless-stopped
    ports:
    - 1688:1688
```
{: file='docker-compose.yml'}

## 激活 Windows 系统
更新 KMS 服务并手动激活系统，以管理员身份执行命令：

```powershell
slmgr /skms x.x.x.x
slmgr /ato
```
`x.x.x.x` 换成 Docker 服务器对应的 IP。
