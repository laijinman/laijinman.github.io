---
layout: post
title: 部署 KMS 服务
date: 2023-11-15 09:33 +0000
categories:
  - 技术
tags: 
  - docker
  - vlmcsd
  - kms
---

通过 docker 方式搭建部署 vlmcsd 应用，实现 KMS（Key Management Service）服务及其使用。

## 部署 KMS 服务

```yml
name: svc

services:
  vlmcsd:
    container_name: vlmcsd
    image: vlmcsd/vlmcsd:${VLMCSD_VERSION:-latest}
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
