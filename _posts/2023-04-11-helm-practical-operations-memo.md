---
layout: post
title: HELM 实用操作备忘录
date: 2023-04-11 06:25 +0000
categories:
- 开发
- 效率
tags:
- kubernetes
- helm
---

[Helm](https://helm.sh/) 是容器环境的包管理器工具，在这里记录一下实际应用中的操作，以备后续查阅。

## 获取包所有版本
Helm 仓库中同一安装包存在多个版本，默认安装为包的最新发布版本。但在实际部署应用中，通常会从包所有版本中选择合适版本进行安装，以确保整体服务的稳定性和兼容性。

获取所有版本操作可以如下操作：
```bash
helm repo update
helm search repo apisix/apisix-dashboard --versions
```
其中`apisix/apisix`是包名（Helm 把包统称`chart`）。

## 自定义包安装
安装包时候，想要了解包支持哪些自定义配置，可以如下操作：
```bash
helm show values apisix/apisix-dashboard --version=0.6.1 \
| tee apisix-dashboard-0.6.1.yml
```
手动修改配置文件`apisix-dashboard-0.6.1.yml`，在安装时指定自定义配置文件，覆盖默认配置达到自定义安装目的。具体操作如下：
```bash
helm install apisix-dashboard apisix/apisix-dashboard \
-f apisix-dashboard-0.6.1.yml \
-n ingress-apisix \
--version=0.6.1
```
